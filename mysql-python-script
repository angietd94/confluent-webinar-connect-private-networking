import mysql.connector
from mysql.connector import errorcode
import random
import time

# MySQL database credentials

config = {
    'user': 'your-rds-username',
    'password': 'your-rds-password',
    'host': 'connect-webinar-database.ck37kmuudyn2.eu-west-1.rds.amazonaws.com',
    'database': 'your-database-name'
}

# Connect to MySQL database
try:
    conn = mysql.connector.connect(**config)
    cursor = conn.cursor()

    # Create customers table
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS customers (
        customer_id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        email VARCHAR(255) NOT NULL
    );
    """)
    print("Customers table created successfully.")

    # Create products table
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS products (
        product_id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        price DECIMAL(10, 2) NOT NULL
    );
    """)
    print("Products table created successfully.")

    # Create orders table
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS orders (
        order_id INT AUTO_INCREMENT PRIMARY KEY,
        customer_id INT,
        product_id INT,
        quantity INT NOT NULL,
        order_status VARCHAR(50) NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
        FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
        FOREIGN KEY (product_id) REFERENCES products(product_id)
    );
    """)
    print("Orders table created successfully.")

    # Create order status history table
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS order_status_history (
        history_id INT AUTO_INCREMENT PRIMARY KEY,
        order_id INT,
        order_status VARCHAR(50) NOT NULL,
        changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        FOREIGN KEY (order_id) REFERENCES orders(order_id)
    );
    """)
    print("Order status history table created successfully.")

    # Insert initial customers
    cursor.executemany("""
    INSERT INTO customers (name, email) VALUES (%s, %s);
    """, [
        ('John Doe', 'john.doe@example.com'),
        ('Jane Smith', 'jane.smith@example.com'),
        ('Alice Johnson', 'alice.johnson@example.com')
    ])
    conn.commit()
    print("Initial customers inserted successfully.")

    # Insert initial products
    cursor.executemany("""
    INSERT INTO products (name, price) VALUES (%s, %s);
    """, [
        ('Laptop', 999.99),
        ('Smartphone', 499.99),
        ('Tablet', 299.99)
    ])
    conn.commit()
    print("Initial products inserted successfully.")

    # Insert initial orders
    cursor.executemany("""
    INSERT INTO orders (customer_id, product_id, quantity, order_status) VALUES (%s, %s, %s, %s);
    """, [
        (1, 1, 1, 'Pending'),
        (2, 2, 2, 'Pending'),
        (3, 3, 1, 'Pending')
    ])
    conn.commit()
    print("Initial orders inserted successfully.")

    # Function to update order status
    def update_order_status(order_id, new_status):
        # Update order status in orders table
        cursor.execute("UPDATE orders SET order_status = %s WHERE order_id = %s;", (new_status, order_id))
        conn.commit()

        # Insert into order status history
        cursor.execute("INSERT INTO order_status_history (order_id, order_status) VALUES (%s, %s);", (order_id, new_status))
        conn.commit()

        print(f"Order {order_id} status updated to {new_status}")

    # List of possible order statuses
    statuses = ['Pending', 'Processed', 'Shipped', 'Delivered']

    # Continuous loop to simulate order status updates
    try:
        while True:
            # Randomly select an order ID and a new status
            order_id = random.randint(1, 3)
            new_status = random.choice(statuses)

            # Update the order status
            update_order_status(order_id, new_status)

            # Wait for a random time interval between 1 and 5 seconds
            time.sleep(random.randint(1, 5))

    except KeyboardInterrupt:
        print("Process interrupted by user.")

except mysql.connector.Error as err:
    if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
        print("Something is wrong with your user name or password")
    elif err.errno == errorcode.ER_BAD_DB_ERROR:
        print("Database does not exist")
    else:
        print(err)
else:
    cursor.close()
    conn.close()
    print("MySQL connection closed.")
