
# Teebay Documentation

Teebay is a product renting and buying/selling application. The application allows users to register, log in, list, add, edit, delete products and rent or buy products.

## Technologies Used
- **Frontend:** React (with Apollo for GraphQL)
- **Backend:** Spring Boot (with GraphQL)
- **Database:** PostgreSQL
- **UI Libraries:** Mantine
- **Forms:** React Hook Form

## DB
### users table
```bash
    CREATE TABLE users (
        id UUID PRIMARY KEY DEFAULT,
        email VARCHAR(255) NOT NULL UNIQUE,
        password VARCHAR(255) NOT NULL,
        user_type VARCHAR(255) NOT NULL, -- Enum: ADMIN, USER
        first_name VARCHAR(255) NOT NULL,
        last_name VARCHAR(255) NOT NULL,
        address VARCHAR(255) NOT NULL,
        phone_number VARCHAR(255) NOT NULL
    );
```

### products table
```bash
    CREATE TABLE products (
        id UUID PRIMARY KEY DEFAULT,
        owner_id UUID NOT NULL,  -- Foreign Key: users(id)
        title VARCHAR(255) NOT NULL,
        description TEXT,
        price DOUBLE PRECISION NOT NULL,
        rent_price DOUBLE PRECISION NOT NULL,
        rent_unit VARCHAR(255) NOT NULL,  -- Enum: HOUR, DAY, WEEK, MONTH
        CONSTRAINT fk_owner FOREIGN KEY (owner_id) REFERENCES users(id),
        CONSTRAINT uk_products_owner_title UNIQUE (owner_id, title)
    );
```

### product_categories table
```bash
    CREATE TABLE product_categories (
        product_id UUID NOT NULL, 
        category VARCHAR(255) NOT NULL,  -- Enum: ELECTRONICS, FURNITURE, HOME_APPLIANCES, SPORTING_GOODS, OUTDOOR, TOYS
        CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(id)
    );
```

### product_order table
```bash
    CREATE TABLE product_order (
        id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
        product_id UUID NOT NULL,  -- Foreign Key: products(id)
        buyer_id UUID NOT NULL,  -- Foreign Key: users(id)
        type VARCHAR(255) NOT NULL,  -- Enum: BUY, RENT
        rent_start TIMESTAMP,
        rent_end TIMESTAMP,
        total_price DOUBLE PRECISION NOT NULL,
        CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(id),
        CONSTRAINT fk_buyer FOREIGN KEY (buyer_id) REFERENCES users(id)
    );
```


## Part 1: Preliminary Features - Login & User Registration
### Login Implementation:
In the frontend, the LOGIN_MUTATION is used to send the email and password to the server. The email field is validated with email test and password is checked if it's length is more than 6. After successful login, the app redirects to the dashboard/myproducts page, which shows the user their own products. 

The login functionality in the backend is implemented by taking the email and password via GraphQl mutation. In the service layer, the email is first checked in the DB if it exists, if not then a BadRequestException is thrown. If the email exists, then password is checked using string matching. If all is matched, then the user id is returned. 

### User Registration:
The frontend has a form which can be reached from the login screen, by clicking the register button. The registration from has email validation test, phone number has to be 11 digits and password has to be more than 6 characters. Each field has null validation to prevent null values to be sent to the backend. The registration form also has confirm password field that matches passwords in the frontend. 

The register backend uses mutation to receive userDto. In the service layer, it checks if the user email already exists in the DB. Each user have to have an unique email. If email doesn't exist, then mapstruct is used to convert the DTO to entity. Since, save to DB can cause unexpected issued, try catch block with InternalErrorException is used to gracefully catch errors. The user has 2 types, USER and ADMIN. For this project only USER type user can be created. 

## Part 2: Product Management (Add, Edit, Delete)
### Product Categories:
Products are classified into Electronics, Furniture, Home Appliances, Sporting goods, Outdoor and Toys. The product form is a multi-page form where users can go back and edit the details before submission.

In the backend, a seprate product category table is maintained with reference to products for the many to many relation.

### Add Product:
The user fills out the product details, including title, category, description, price, and rental information. Each field has null checks and corresponding error message in the frontend. After validation, the data is submitted to the backend through GraphQl mutation.

The backend receives the productDto and checks if the owner of the product exists. The userId is taken from the authentication interceptor to take the userId from request header. Afterward, using mapstruct dto is converted to entity to be saved in the DB. Try catch and graceful exception handling is used when saving to db via repository.

### Edit Product
Users can edit their own product details. The updated information is sent to the backend, which updates the database. The Apollo cache is updated to reflect the changes immediately in the frontend.

The backend checks if the product exists, verifies the owner if they exist. The userId is taken from the authentication interceptor to take the userId from request header. Beancopy is used to copy the updated product data to the existing products. Afterwards, the updated existing product is saved gracefull with try catch and excpetion handling.

### Delete Product:
When a user deletes a product, the backend removes it from the database, and Apollo cache is cleared to ensure the ui is updated.

The backend has product and owner data existance checks with same interceptor to catch header auth. This also has try catch and graceful exception handling before delteing via repository.

## Part 3: Rent and Buy/Sell Products
### Product Listing:
All products from all users are listed. The frontend fetches this list using GraphQL queries, and the data is stored in the Apollo in-memory cache. The dashboard's landing page is a list of user's products. A hamburger menu is used to swtich between all products, user's products and history. History page holds the user's transactions.

### Buy Product:
When a user chooses to buy a product from the product list, status is updated in Apollo cache for immediate ui reflection. 

In the backend, it checks if the user type is USER as ADMIN user should not be able to buy or rent products. Then it checks if the product exists and checks if the user owns the product as user can not buy or rent their own products. Later on, total price is calculated. For buy product case, the calculation is simple as it takes the product's price. Then a order entity is created with appropriate data and saved gracefully.

### Rent Product:
For rentals, users can specify the duration for renting for a product. If a rent overlapping occours, an error message is displayed. If a user rents a product, the Apollo cache is updated to reflect the new rental status.

The backend reuses the buy product method in the service layer but for rentals, a rent overlap validation is done in the repository layer. Total rent price is calculated using the start and end dates. 

### Display Bought/Sold/Borrowed/Lent Products:
Users can view all the products they have bought, sold, rented, or lent in the history page from the dashboard's hamburger menu. This information is retrieved from the database via GraphQl queries and displayed on the user's dashboard in separate tabs.

In the backend, the data fetching is done on the repository layer.

## Part 4: Corner Cases and Problem-Solving
### Handling Overlapping Rentals:
If two users attempt to rent the same product for overlapping dates, the system checks the available period before allowing the transaction. If a conflict occurs, an error message is displayed. This is done in the backend using repository layer.

### Getting GraphQL to function as intended
Getting graphQl to function properly in the backend requires strongly typed schema that has to match with the mutations and queries.

### Error handling
To handle errors gracefully and smooth front end error showing, custom error exceptions is created that extends runtime exception in the backend.

### Apollo cache retention
When adding new product then redirecting to user's products, all products and after buying/renting the history page needs to be updated accordingly. If apollo cache is not updated, a hard reload is required.  