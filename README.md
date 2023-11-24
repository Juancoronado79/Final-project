import random
import pytest
import logging


class BankAccount:
    def __init__(self, account_number, balance=0):
        self.account_number = account_number
        self.balance = balance


class Customer:
    def __init__(self, name, address, phone_number, username, password, initial_balance):
        self.name = name
        self.address = address
        self.phone_number = phone_number
        self.username = username
        self.password = password
        self.accounts = [BankAccount(random.randint(1000, 9999), initial_balance)]

    def display_accounts(self):
        for account in self.accounts:
            print(f"Account Number: {account.account_number}, Balance: ${account.balance}")

    def display_balance(self):
        if len(self.accounts) == 1:
            print(f"Your currently balance is ${self.accounts[0].balance}")
        else:
            print("Choose an account:")
            self.display_accounts()
            account_number = int(input("Please enter Account Number: "))
            selected_account = next((account for account in self.accounts if account.account_number == account_number), None)
            if selected_account:
                print(f"Your currently balance is ${selected_account.balance}")
            else:
                print("Invalid account number.")

    def withdraw(self):
        if len(self.accounts) == 1:
            account = self.accounts[0]
            amount = float(input("Please enter the amount to withdraw: "))
            if 0 < amount <= account.balance:
                account.balance -= amount
                print(f"Withdrawal successful. New balance: ${account.balance}")
            else:
                print("Invalid amount or insufficient balance.")
        else:
            print("Please select an account:")
            self.display_accounts()
            account_number = int(input("Please enter Account Number: "))
            selected_account = next((account for account in self.accounts if account.account_number == account_number), None)
            if selected_account:
                amount = float(input("please enter the amount to withdraw: "))
                if 0 < amount <= selected_account.balance:
                    selected_account.balance -= amount
                    print(f"Withdrawal successful. New balance: ${selected_account.balance}")
                else:
                    print("Invalid amount or insufficient balance.")
            else:
                print("Invalid account number.")

    def deposit(self):
        if len(self.accounts) == 1:
            account = self.accounts[0]
            amount = float(input("Please enter the amount to deposit: "))
            if amount > 0:
                account.balance += amount
                print(f"Deposit successful. New balance: ${account.balance}")
            else:
                print("Invalid amount.")
        else:
            print("Select an account:")
            self.display_accounts()
            account_number = int(input("Please enter Account Number: "))
            selected_account = next((account for account in self.accounts if account.account_number == account_number), None)
            if selected_account:
                amount = float(input("Please enter the amount to deposit: "))
                if amount > 0:
                    selected_account.balance += amount
                    print(f"Deposit successful. New balance: ${selected_account.balance}")
                else:
                    print("Invalid amount.")
            else:
                print("Invalid account number.")


class Admin:
    def __init__(self, username, password):
        self.username = username
        self.password = password

    def view_customer_info(self, customers):
        for customer in customers:
            print(f"Name: {customer.name}, Address: {customer.address}, Phone: {customer.phone_number}")

    def delete_customer(self, customers):
        username = input("Enter the username of the customer to delete: ")
        customers[:] = [customer for customer in customers if customer.username != username]
        print(f"Customer with username {username} deleted.")

    def update_customer_info(self, customers):
        username = input("Enter the username of the customer to update: ")
        customer = next((customer for customer in customers if customer.username == username), None)

        if customer:
            print(f"Updating information for customer: {customer.name}")
            customer.name = input("New name: ")
            customer.address = input("New address: ")
            customer.phone_number = input("New phone number: ")
            print("Customer information updated.")
        else:
            print("Customer not found.")


class BankSystem:
    def __init__(self):
        self.customers = []
        self.admins = [Admin("admin", "admin")]

    def run(self):
        logging.basicConfig(filename='bank_system.log', level=logging.INFO, format='%(asctime)s - %(message)s')
        logging.info("Bank system started.")

        print("Welcome to coronado Bank!")
        while True:
            print("\nOptions:")
            print("1. Customer")
            print("2. Administrator")
            print("3. Exit")

            choice = input("Select an option (1/2/3): ")

            if choice == "1":
                self.customer_menu()
            elif choice == "2":
                self.admin_menu()
            elif choice == "3":
                logging.info("Bank system closed.")
                print("Thank you for using our system. Goodbye!")
                break
            else:
                print("Invalid option. Please select again.")

    def customer_menu(self):
        print("\nCustomer Options:")
        print("1. Create new login")
        print("2. Existing customer")

        customer_choice = input("Select an option (1/2): ")

        if customer_choice == "1":
            self.create_customer()
        elif customer_choice == "2":
            self.existing_customer_login()
        else:
            print("Invalid option. Please select again.")

    def create_customer(self):
        name = input("Enter your name: ")
        address = input("Enter your address: ")
        phone_number = input("Enter your phone number: ")
        username = input("Enter a username: ")
        password = input("Enter a password: ")
        initial_balance = float(input("Enter the initial balance: "))

        new_customer = Customer(name, address, phone_number, username, password, initial_balance)
        self.customers.append(new_customer)
        logging.info(f"New customer created: {new_customer.name}")
        print("Registration successful!")

    def existing_customer_login(self):
        username = input("Enter your username: ")
        password = input("Enter your password: ")

        customer = next((c for c in self.customers if c.username == username and c.password == password), None)

        if customer:
            logging.info(f"Successful login for customer: {customer.name}")
            self.customer_page(customer)
        else:
            logging.warning("Login failed. Check your username and password.")

    def customer_page(self, customer):
        print(f"Welcome, {customer.name}!")
        while True:
            print("\nOptions:")
            print("1. Create new bank account")
            print("2. List bank accounts")
            print("3. Display balance")
            print("4. Withdraw")
            print("5. Deposit")
            print("6. Logout")

            customer_choice = input("Select an option (1/2/3/4/5/6): ")

            if customer_choice == "1":
                self.create_bank_account(customer)
            elif customer_choice == "2":
                customer.display_accounts()
            elif customer_choice == "3":
                customer.display_balance()
            elif customer_choice == "4":
                customer.withdraw()
            elif customer_choice == "5":
                customer.deposit()
            elif customer_choice == "6":
                logging.info(f"Logout for customer: {customer.name}")
                print(f"Goodbye, {customer.name}!")
                break
            else:
                print("Invalid option. Please select again.")

    def create_bank_account(self, customer):
        new_account = BankAccount(random.randint(1000, 9999))
        while any(account.account_number == new_account.account_number for account in customer.accounts):
            new_account = BankAccount(random.randint(1000, 9999))
        customer.accounts.append(new_account)
        logging.info(f"New account created for customer: {customer.name}, Account number: {new_account.account_number}")
        print(f"A new account has been created with the number {new_account.account_number}.")

    def admin_menu(self):
        admin_username = input("Enter the administrator username: ")
        admin_password = input("Enter the administrator password: ")

        admin = next((a for a in self.admins if a.username == admin_username and a.password == admin_password), None)

        if admin:
            print("Successful administrator login.")
            while True:
                print("\nAdministrator Options:")
                print("1. View information for all customers")
                print("2. Delete a customer")
                print("3. Update information for a customer")
                print("4. Logout")

                admin_choice = input("Select an option (1/2/3/4): ")

                if admin_choice == "1":
                    admin.view_customer_info(self.customers)
                elif admin_choice == "2":
                    admin.delete_customer(self.customers)
                elif admin_choice == "3":
                    admin.update_customer_info(self.customers)
                elif admin_choice == "4":
                    logging.info("Administrator logout.")
                    print("Goodbye, administrator!")
                    break
                else:
                    print("Invalid option. Please select again.")
        else:
            logging.warning("Administrator login failed. Check the username and password.")


# Unit tests with Pytest
def test_create_customer():
    bank_system = BankSystem()
    bank_system.create_customer("John Doe", "123 Main St", "555-1234", "john_doe", "password123", 1000)
    assert len(bank_system.customers) == 1


def test_existing_customer_login():
    bank_system = BankSystem()
    bank_system.create_customer("Jane Doe", "456 Oak St", "555-5678", "jane_doe", "password456", 1500)
    customer = bank_system.existing_customer_login("jane_doe", "password456")
    assert customer is not None

    invalid_customer = bank_system.existing_customer_login("nonexistent_user", "wrong_password")
    assert invalid_customer is None


# Instance of the bank system
bank_system = BankSystem()
# Run the application
bank_system.run()# Final-project
Banking system
