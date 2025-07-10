# data-sync-monitor

Creating a `data-sync-monitor` to ensure data consistency and integrity between distributed databases involves multiple steps. This will include connecting to multiple databases, comparing data, and logging discrepancies or other issues. For this example, I'll create a simplified program using SQLite databases for demonstration purposes. In a real-world scenario, you might be working with SQL Server, MySQL, PostgreSQL, etc.

Here's a Python program to achieve basic synchronization monitoring:

```python
import sqlite3
from sqlite3 import Error

def create_connection(db_file):
    """Create a database connection to the provided SQLite database."""
    conn = None
    try:
        conn = sqlite3.connect(db_file)
        print(f"Connected to database: {db_file}")
    except Error as e:
        print(f"Error connecting to database {db_file}: {e}")
    return conn

def fetch_data(conn, table_name):
    """Fetch all data from the specified table."""
    try:
        cur = conn.cursor()
        cur.execute(f"SELECT * FROM {table_name}")
        rows = cur.fetchall()
        return rows
    except Error as e:
        print(f"Error fetching data from table {table_name}: {e}")
        return None

def compare_data(data1, data2):
    """Compare two datasets and return discrepancies."""
    # Convert data to sets of tuples for comparison
    set1 = set(data1)
    set2 = set(data2)
    
    # Find discrepancies
    in_first_not_second = set1 - set2  # Present in first but not in second
    in_second_not_first = set2 - set1  # Present in second but not in first

    return in_first_not_second, in_second_not_first

def sync_monitor(db1_path, db2_path, table_name):
    """Monitor data synchronization between two given databases."""
    # Create database connections
    conn1 = create_connection(db1_path)
    conn2 = create_connection(db2_path)

    if conn1 is not None and conn2 is not None:
        # Fetch data
        data1 = fetch_data(conn1, table_name)
        data2 = fetch_data(conn2, table_name)

        if data1 is not None and data2 is not None:
            # Compare data
            discrepancies1, discrepancies2 = compare_data(data1, data2)

            # Log discrepancies
            if discrepancies1 or discrepancies2:
                print("Discrepancies found between databases:")
                if discrepancies1:
                    print("\nRecords in the first database but not in the second:")
                    for record in discrepancies1:
                        print(record)

                if discrepancies2:
                    print("\nRecords in the second database but not in the first:")
                    for record in discrepancies2:
                        print(record)
            else:
                print("No discrepancies found. Data is consistent.")
        else:
            print("Error in fetching data from one or both databases.")
    else:
        print("Error in establishing database connections.")

    # Close connections
    if conn1:
        conn1.close()
    if conn2:
        conn2.close()

# Example usage:
# For testing purposes, the SQLite databases should have initialized with the same structure.
# Create two SQLite databases: 'db1.sqlite' and 'db2.sqlite’ with a table 'example_table'
# containing some sample data for comparison.
if __name__ == "__main__":
    db1_path = "db1.sqlite"
    db2_path = "db2.sqlite"
    table_name = "example_table"
    sync_monitor(db1_path, db2_path, table_name)
```

### Important Points:

1. **Database Setup**: Before running the program, ensure you have two SQLite database files (`db1.sqlite`, `db2.sqlite`) with the same table structure and a few records to test the logic.

2. **Error Handling**: The code includes error handling for database connection issues and data fetching errors.

3. **Comparison Logic**: The program uses Python sets to find differences between the datasets from each database.

4. **Logging**: The program logs any discrepancies found between the databases. This is crucial for debugging and keeping track of data integrity issues.

For a production-level tool, consider adding more robust logging, comprehensive error handling, support for different database types, and perhaps a user interface or API for easier operation.