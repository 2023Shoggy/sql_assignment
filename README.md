!pip install faker
import sqlite3
import numpy as np
from faker import Faker

# Initialize faker
fake = Faker()
# Function to create a connection to the SQLite database
def create_connection(db_file):
    conn = None
    try:
        conn = sqlite3.connect(db_file)
        print(f"Connected to database: {db_file}")
        return conn
    except sqlite3.Error as e:
        print(e)
    return conn
    # Function to create tables in the database
def create_tables(conn):
    try:
        cursor = conn.cursor()

        # Create Car Manufacturers table
        cursor.execute('''CREATE TABLE IF NOT EXISTS CarManufacturers (
                            ManufacturerID INTEGER PRIMARY KEY,
                            ManufacturerName TEXT
                            )''')

        # Create Car Models table
        cursor.execute('''CREATE TABLE IF NOT EXISTS CarModels (
                            ModelID INTEGER PRIMARY KEY,
                            ManufacturerID INTEGER,
                            ModelName TEXT,
                            FOREIGN KEY (ManufacturerID) REFERENCES CarManufacturers(ManufacturerID)
                            )''')

        # Create Car Wheels table
        cursor.execute('''CREATE TABLE IF NOT EXISTS CarWheels (
                            WheelID INTEGER PRIMARY KEY,
                            ModelID INTEGER,
                            WheelType TEXT,
                            WheelSize INTEGER,
                            FOREIGN KEY (ModelID) REFERENCES CarModels(ModelID)
                            )''')

        # Create Car Sales table
        cursor.execute('''CREATE TABLE IF NOT EXISTS CarSales (
                            SaleID INTEGER PRIMARY KEY,
                            ModelID INTEGER,
                            SaleDate TEXT,
                            SalePrice REAL,
                            BuyerName TEXT,
                            BuyerContact TEXT,
                            FOREIGN KEY (ModelID) REFERENCES CarModels(ModelID)
                            )''')

        # Create Car Reviews table
        cursor.execute('''CREATE TABLE IF NOT EXISTS CarReviews (
                            ReviewID INTEGER PRIMARY KEY,
                            ModelID INTEGER,
                            ReviewerName TEXT,
                            ReviewDate TEXT,
                            ReviewText TEXT,
                            Rating INTEGER,
                            FOREIGN KEY (ModelID) REFERENCES CarModels(ModelID)
                            )''')

        # Create Car Maintenance Records table
        cursor.execute('''CREATE TABLE IF NOT EXISTS CarMaintenanceRecords (
                            MaintenanceID INTEGER PRIMARY KEY,
                            ModelID INTEGER,
                            MaintenanceDate TEXT,
                            MaintenanceType TEXT,
                            MaintenanceDescription TEXT,
                            Cost REAL,
                            FOREIGN KEY (ModelID) REFERENCES CarModels(ModelID)
                            )''')

        print("Tables created successfully.")
        conn.commit()
    except sqlite3.Error as e:
        print(e)
        conn.rollback()


# Function to generate sample data for tables
def generate_sample_data(conn, num_records):
    try:
        cursor = conn.cursor()

        # Generate sample data for Car Manufacturers
        manufacturers = [(None, fake.company()) for _ in range(num_records)]
        cursor.executemany("INSERT INTO CarManufacturers VALUES (?, ?)", manufacturers)

        # Generate sample data for Car Models
        models = [(None, np.random.randint(1, num_records+1), fake.word()) for _ in range(num_records)]
        cursor.executemany("INSERT INTO CarModels VALUES (?, ?, ?)", models)

        # Generate sample data for Car Wheels
        wheels = [(None, np.random.randint(1, num_records+1), fake.word(), np.random.randint(15, 20)) for _ in range(num_records)]
        cursor.executemany("INSERT INTO CarWheels VALUES (?, ?, ?, ?)", wheels)

        # Generate sample data for Car Sales
        sales = [(None, np.random.randint(1, num_records+1), fake.date_this_decade(), np.random.randint(10000, 50000), fake.name(), fake.phone_number()) for _ in range(num_records)]
        cursor.executemany("INSERT INTO CarSales VALUES (?, ?, ?, ?, ?, ?)", sales)

        # Generate sample data for Car Reviews
        reviews = [(None, np.random.randint(1, num_records+1), fake.name(), fake.date_this_decade(), fake.paragraph(), np.random.randint(1, 6)) for _ in range(num_records)]
        cursor.executemany("INSERT INTO CarReviews VALUES (?, ?, ?, ?, ?, ?)", reviews)

        # Generate sample data for Car Maintenance Records
        maintenance_records = [(None, np.random.randint(1, num_records+1), fake.date_this_decade(), fake.word(), fake.sentence(), np.random.randint(50, 500)) for _ in range(num_records)]
        cursor.executemany("INSERT INTO CarMaintenanceRecords VALUES (?, ?, ?, ?, ?, ?)", maintenance_records)

        print("Sample data generated successfully.")
        conn.commit()
    except sqlite3.Error as e:
        print(e)
        conn.rollback()

        # Main function to run the script
database = "vehicle_database.db"
conn = create_connection(database)
if conn is not None:
  create_tables(conn)
  generate_sample_data(conn, 500)
  conn.close()
else:
    print("Error! Cannot create the database connection.")
