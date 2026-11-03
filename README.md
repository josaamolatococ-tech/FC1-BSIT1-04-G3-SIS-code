import os

MIN_USERNAME_LENGTH = 3
MIN_PASSWORD_LENGTH = 4

usernames = ["gwapong_manager", "gwapangastaff"]
passwords = ["passwordsagwapo", "passwordsagwapa"]
roles = ["Manager", "Staff"]


def check_credentials(username, password):
    for i in range(len(usernames)):
        if usernames[i] == username and passwords[i] == password:
            return roles[i]
    return ""


def find_account_position(username):
    for i in range(len(usernames)):
        if usernames[i] == username:
            return i
    return -1


def username_exists(username):
    for name in usernames:
        if name.lower() == username.lower():
            return True
    return False


def get_staff_usernames():
    staff_list = []
    for i in range(len(usernames)):
        if roles[i] == "Staff":
            staff_list.append(usernames[i])
    return staff_list


def add_staff_account(username, password):
    usernames.append(username)
    passwords.append(password)
    roles.append("Staff")


def reset_password(username, new_password):
    position = find_account_position(username)
    if position == -1:
        return False
    passwords[position] = new_password
    return True


def remove_account(username):
    position = find_account_position(username)
    if position == -1:
        return False
    usernames.pop(position)
    passwords.pop(position)
    roles.pop(position)
    return True


LOW_STOCK_PERCENT = 20
DEFAULT_FULL_STOCK = 100
MAX_LOGIN_ATTEMPTS = 3
MAX_NAME_LENGTH = 22
LINE_WIDTH = 83

inventory = []
categories = ["Beverages", "Snacks", "Rice Meals", "Bread & Pastries",
              "Ingredients", "Disposables"]

sample_items = [
    ["Bottled Water 500ml", "Beverages", 48],
    ["Royal Mismo 200ml", "Beverages", 8],
    ["Coke Mismo 200ml", "Beverages", 25],
    ["Sprite Mismo 200ml", "Beverages", 18],
    ["C2 Apple 230ml", "Beverages", 30],
    ["Zesto Orange Juice", "Beverages", 42],
    ["Milo Sachet", "Beverages", 0],
    ["Nescafe 3-in-1 Sachet", "Beverages", 55],
    ["Piattos Cheese", "Snacks", 32],
    ["Chippy BBQ", "Snacks", 6],
    ["Nova Cheese", "Snacks", 40],
    ["Skyflakes Crackers", "Snacks", 36],
    ["Fita Biscuit", "Snacks", 0],
    ["Lucky Me Pancit Canton", "Snacks", 24],
    ["Boiled Peanuts Pack", "Snacks", 15],
    ["Chicken Adobo w/ Rice", "Rice Meals", 20],
    ["Pork Sinigang w/ Rice", "Rice Meals", 35],
    ["Tapsilog", "Rice Meals", 14],
    ["Hotsilog", "Rice Meals", 28],
    ["Pork Menudo w/ Rice", "Rice Meals", 12],
    ["Pandesal Pack", "Bread & Pastries", 60],
    ["Malunggay Pandesal", "Bread & Pastries", 35],
    ["Ensaymada", "Bread & Pastries", 9],
    ["Spanish Bread", "Bread & Pastries", 22],
    ["Pan de Coco", "Bread & Pastries", 16],
    ["Rice (25kg sack)", "Ingredients", 3],
    ["Cooking Oil 1L", "Ingredients", 10],
    ["Eggs (Tray of 30)", "Ingredients", 6],
    ["Soy Sauce 1L", "Ingredients", 8],
    ["Vinegar 1L", "Ingredients", 9],
    ["Tender Juicy Hotdog", "Ingredients", 45],
    ["Paper Cups", "Disposables", 60],
    ["Plastic Spoon Pack", "Disposables", 7],
    ["Styro Meal Box", "Disposables", 80],
    ["Plastic Straw Pack", "Disposables", 25],
    ["Plastic Labo (Small)", "Disposables", 0],
]


TAG_ERROR = "[ERROR]"
TAG_SUCCESS = "[SUCCESS]"
TAG_ALERT = "[ALERT]"
TAG_CANCELLED = "[CANCELLED]"
TAG_LOCKED = "[LOCKED]"


def clear_screen():
    os.system("cls" if os.name == "nt" else "clear")


def pause(message="Press Enter to continue..."):
    input("\n  " + message)


def make_bar(part, whole, width=20):
    if whole == 0:
        filled = 0
    else:
        filled = round(part / whole * width)
    return "[" + "#" * filled + "." * (width - filled) + "]"


def compute_alert_level(full_stock):
    return full_stock * LOW_STOCK_PERCENT / 100


def get_status(quantity, full_stock):
    if quantity == 0:
        return "Out of Stock"
    elif quantity <= compute_alert_level(full_stock):
        return "Low Stock"
    else:
        return "In Stock"


def compute_total_units():
    total = 0
    for record in inventory:
        total += record[3]
    return total


def count_by_status(status):
    count = 0
    for record in inventory:
        if record[4] == status:
            count += 1
    return count


def compute_category_total(category):
    total = 0
    for record in inventory:
        if record[2] == category:
            total += record[3]
    return total


def compute_restock_amount(record):
    amount = record[5] - record[3]
    if amount < 0:
        amount = 0
    return amount


def get_low_stock_items():
    low_items = []
    for record in inventory:
        if record[4] == "Low Stock" or record[4] == "Out of Stock":
            low_items.append(record)
    return low_items


def compute_total_restock():
    total = 0
    for record in get_low_stock_items():
        total += compute_restock_amount(record)
    return total


def compute_percentage(part, whole):
    if whole == 0:
        return 0.0
    return round(part / whole * 100, 1)


def find_item(item_id):
    for record in inventory:
        if record[0] == item_id:
            return record
    return None


def name_exists(item_name):
    for record in inventory:
        if record[1].lower() == item_name.lower():
            return True
    return False


def generate_item_id():
    highest = 1000
    for record in inventory:
        if record[0] > highest:
            highest = record[0]
    return highest + 1


def add_item(item_name, category, quantity, full_stock):
    record = [generate_item_id(), item_name, category, quantity,
              get_status(quantity, full_stock), full_stock]
    inventory.append(record)
    return record


def change_quantity(record, new_quantity):
    record[3] = new_quantity
    record[4] = get_status(new_quantity, record[5])


def delete_item(record):
    inventory.remove(record)


def load_sample_items():
    for sample in sample_items:
        add_item(sample[0], sample[1], sample[2], DEFAULT_FULL_STOCK)


def get_menu_options(role):
    if role == "Manager":
        return [
            ["view", "View All Items", "INVENTORY"],
            ["add", "Add New Item", "INVENTORY"],
            ["stock_in", "Stock In (Delivery)", "INVENTORY"],
            ["stock_out", "Stock Out (Sold / Used)", "INVENTORY"],
            ["count", "Physical Count (Adjust Stock)", "INVENTORY"],
            ["delete", "Delete Item", "INVENTORY"],
            ["low", "Low Stock Report", "REPORTS"],
            ["summary", "Inventory Summary Report", "REPORTS"],
            ["accounts", "Manage Staff Accounts", "ADMINISTRATION"],
            ["logout", "Log Out", "SESSION"],
            ["exit", "Exit System", "SESSION"],
        ]
    else:
        return [
            ["view", "View All Items", "INVENTORY"],
            ["stock_in", "Stock In (Delivery)", "INVENTORY"],
            ["stock_out", "Stock Out (Sold / Used)", "INVENTORY"],
            ["low", "Low Stock Report", "REPORTS"],
            ["logout", "Log Out", "SESSION"],
            ["exit", "Exit System", "SESSION"],
        ]


def read_text(prompt):
    text = input(prompt).strip()
    while text == "":
        print(f"  {TAG_ERROR} Input cannot be empty.")
        text = input(prompt).strip()
    return text


def read_number(prompt, minimum):
    text = input(prompt).strip()
    while not text.isdigit() or int(text) < minimum:
        print(f"  {TAG_ERROR} Please enter a whole number "
              f"(minimum {minimum}).")
        text = input(prompt).strip()
    return int(text)


def read_menu_choice(prompt, total_options):
    choice = input(prompt).strip()
    while not choice.isdigit() or not 1 <= int(choice) <= total_options:
        print(f"\n  {TAG_ERROR} Invalid option. "
              f"Please enter 1 to {total_options}.")
        choice = input(prompt).strip()
    return int(choice)


def print_title(title, subtitle=""):
    inner = LINE_WIDTH - 2
    print("\n" + "╔" + "═" * inner + "╗")
    print("║" + title.center(inner) + "║")
    if subtitle != "":
        print("╠" + "═" * inner + "╣")
        print("║" + subtitle.center(inner) + "║")
    print("╚" + "═" * inner + "╝")


def print_section_header(title):
    print("\n" + "─" * LINE_WIDTH)
    print(f" ▌ {title}")
    print("─" * LINE_WIDTH)


def print_line(char="─"):
    print(char * LINE_WIDTH)


def choose_category():
    print("\n  Select a category:")
    for i in range(len(categories)):
        print(f"    [{i + 1}] {categories[i]}")
    choice = read_number("  Enter category number: ", 1)
    while choice > len(categories):
        print(f"  {TAG_ERROR} Category number is not on the list.")
        choice = read_number("  Enter category number: ", 1)
    return categories[choice - 1]


def pick_item(prompt):
    if len(inventory) == 0:
        print(f"  {TAG_ERROR} The inventory is empty. Add an item first.")
        return None
    item_id = read_number(prompt, 0)
    record = find_item(item_id)
    if record is None:
        print(f"  {TAG_ERROR} Item ID not found!")
    return record


def read_new_username():
    username = read_text("  New username: ")
    while (len(username) < MIN_USERNAME_LENGTH
           or " " in username
           or username_exists(username)):
        if len(username) < MIN_USERNAME_LENGTH or " " in username:
            print(f"  {TAG_ERROR} Username needs "
                  f"{MIN_USERNAME_LENGTH}+ "
                  f"characters and no spaces.")
        else:
            print(f"  {TAG_ERROR} That username is already taken.")
        username = read_text("  New username: ")
    return username


def read_new_password(prompt):
    password = read_text(prompt)
    while len(password) < MIN_PASSWORD_LENGTH:
        print(f"  {TAG_ERROR} Password needs at least "
              f"{MIN_PASSWORD_LENGTH} characters.")
        password = read_text(prompt)
    return password


def display_menu(username, role, options):
    print_title("SCHOOL CANTEEN INVENTORY MONITORING SYSTEM",
                f"MAIN MENU  |  {username}  ({role})")
    last_section = ""
    for i in range(len(options)):
        if options[i][2] != last_section:
            last_section = options[i][2]
            print(f"\n  ■ {last_section}")
        print(f"     [{i + 1:>2}] {options[i][1]}")
    print("\n" + "═" * LINE_WIDTH)


def show_stock_alert(record):
    if record[4] == "Low Stock":
        print(f"  {TAG_ALERT} Low stock! '{record[1]}' has only "
              f"{record[3]} left. "
              f"Suggested restock: {compute_restock_amount(record)} "
              f"units.")
    elif record[4] == "Out of Stock":
        print(f"  {TAG_ALERT} '{record[1]}' is OUT OF STOCK! "
              f"Suggested restock: {compute_restock_amount(record)} "
              f"units.")


def display_all_items():
    print_title("CURRENT CANTEEN STOCK LIST")
    if len(inventory) == 0:
        print(" No items recorded yet.")
    else:
        header = (f"{'ID':<6} │ {'ITEM NAME':<22} │ {'CATEGORY':<18} │ "
                  f"{'QTY':>5} │ {'FULL':>5} │ {'STATUS':<12}")
        print(header)
        print("─" * LINE_WIDTH)
        for record in inventory:
            print(f"{record[0]:<6} │ {record[1]:<22} │ {record[2]:<18} │ "
                  f"{record[3]:>5} │ {record[5]:>5} │ {record[4]:<12}")
    print("═" * LINE_WIDTH)


def display_low_stock_report():
    low_items = get_low_stock_items()
    print_title("LOW STOCK ALERT REPORT")
    if len(low_items) == 0:
        print(" All items are sufficiently stocked.")
    else:
        header = (f"{'ID':<6} │ {'ITEM NAME':<22} │ {'QTY':>5} │ "
                  f"{'FULL':>5} │ {'STATUS':<12} │ {'RESTOCK':>8}")
        print(header)
        print("─" * LINE_WIDTH)
        for record in low_items:
            restock = compute_restock_amount(record)
            print(f"{record[0]:<6} │ {record[1]:<22} │ {record[3]:>5} │ "
                  f"{record[5]:>5} │ {record[4]:<12} │ {restock:>8}")
        print("─" * LINE_WIDTH)
        print(f" Items needing restock: {len(low_items)}")
    print("═" * LINE_WIDTH)


def display_summary_report():
    total_items = len(inventory)
    in_stock = count_by_status("In Stock")
    low_stock = count_by_status("Low Stock")
    out_stock = count_by_status("Out of Stock")
    print_title("INVENTORY SUMMARY REPORT")
    print(f" Total item records  : {total_items}")
    print(f" Total units in stock: {compute_total_units()}")
    print_line()
    print(f" In Stock    : {in_stock:>3} item(s) "
          f"({compute_percentage(in_stock, total_items)}%)")
    print("   " + make_bar(in_stock, total_items))
    print(f" Low Stock   : {low_stock:>3} item(s) "
          f"({compute_percentage(low_stock, total_items)}%)")
    print("   " + make_bar(low_stock, total_items))
    print(f" Out of Stock: {out_stock:>3} item(s) "
          f"({compute_percentage(out_stock, total_items)}%)")
    print("   " + make_bar(out_stock, total_items))
    print_line()
    print(" Units per category:")
    for category in categories:
        print(f"   {category:<20}: {compute_category_total(category):>5}")
    print_line()
    print(f" Total units needed to restock: {compute_total_restock()}")
    print("═" * LINE_WIDTH)


def display_staff_accounts():
    staff_list = get_staff_usernames()
    print_section_header("STAFF ACCOUNTS")
    if len(staff_list) == 0:
        print(" No staff accounts yet.")
    else:
        for i in range(len(staff_list)):
            print(f"  [{i + 1}] {staff_list[i]}")
    print_line()


def handle_add_item():
    print_section_header("ADD NEW ITEM")
    item_name = read_text("  Enter item name: ")
    while len(item_name) > MAX_NAME_LENGTH or name_exists(item_name):
        if len(item_name) > MAX_NAME_LENGTH:
            print(f"  {TAG_ERROR} Name must be {MAX_NAME_LENGTH} characters "
                  f"or less.")
        else:
            print(f"  {TAG_ERROR} Item already exists. "
                  f"Use Stock In instead.")
        item_name = read_text("  Enter item name: ")
    category = choose_category()
    quantity = read_number("  Enter quantity (this is the full stock): ", 1)
    record = add_item(item_name, category, quantity, quantity)
    alert_level = int(compute_alert_level(record[5]))
    print(f"\n  {TAG_SUCCESS} Item added: {record[1]} (ID: {record[0]}) "
          f"- Status: {record[4]}")
    print(f"  Full stock: {record[5]} | Low stock alert at {alert_level} "
          f"or less ({LOW_STOCK_PERCENT}%)")


def handle_stock_in():
    print_section_header("STOCK IN (DELIVERY)")
    record = pick_item("  Enter Item ID: ")
    if record is None:
        return
    print(f"  Selected: {record[1]} | Quantity: {record[3]} "
          f"({record[4]})")
    amount = read_number("  Enter quantity received: ", 1)
    old_quantity = record[3]
    change_quantity(record, old_quantity + amount)
    print(f"\n  {TAG_SUCCESS} {record[1]}: {old_quantity} -> {record[3]} "
          f"({record[4]})")


def handle_stock_out():
    print_section_header("STOCK OUT (SOLD / USED)")
    record = pick_item("  Enter Item ID: ")
    if record is None:
        return
    print(f"  Selected: {record[1]} | Quantity: {record[3]} "
          f"({record[4]})")
    amount = read_number("  Enter quantity sold/used: ", 1)
    if amount > record[3]:
        print(f"  {TAG_ERROR} Cannot remove {amount}. "
              f"Only {record[3]} in stock.")
        return
    old_quantity = record[3]
    change_quantity(record, old_quantity - amount)
    print(f"\n  {TAG_SUCCESS} {record[1]}: {old_quantity} -> {record[3]} "
          f"({record[4]})")
    show_stock_alert(record)


def handle_physical_count():
    print_section_header("PHYSICAL COUNT (ADJUST STOCK)")
    record = pick_item("  Enter Item ID: ")
    if record is None:
        return
    print(f"  Selected: {record[1]} | System quantity: {record[3]} "
          f"({record[4]})")
    new_quantity = read_number("  Enter the counted quantity: ", 0)
    old_quantity = record[3]
    change_quantity(record, new_quantity)
    print(f"\n  {TAG_SUCCESS} {record[1]}: {old_quantity} -> {record[3]} "
          f"({record[4]})")
    show_stock_alert(record)


def handle_delete_item():
    print_section_header("DELETE ITEM")
    record = pick_item("  Enter Item ID to delete: ")
    if record is None:
        return
    confirm = input(f"  Delete '{record[1]}'? (Y/N): ").strip().lower()
    if confirm == "y":
        delete_item(record)
        print(f"\n  {TAG_SUCCESS} '{record[1]}' was removed from the "
              f"inventory.")
    else:
        print(f"\n  {TAG_CANCELLED} No changes were made.")


def pick_staff_account():
    staff_list = get_staff_usernames()
    if len(staff_list) == 0:
        print(f"  {TAG_ERROR} There are no staff accounts.")
        return None
    display_staff_accounts()
    number = read_number("  Enter staff number: ", 1)
    if number > len(staff_list):
        print(f"  {TAG_ERROR} Staff number is not on the list.")
        return None
    return staff_list[number - 1]


def handle_add_staff():
    print_section_header("ADD STAFF ACCOUNT")
    username = read_new_username()
    password = read_new_password("  New password: ")
    add_staff_account(username, password)
    print(f"\n  {TAG_SUCCESS} Staff account '{username}' created.")


def handle_reset_password():
    print_section_header("RESET STAFF PASSWORD")
    username = pick_staff_account()
    if username is None:
        return
    password = read_new_password("  New password: ")
    reset_password(username, password)
    print(f"\n  {TAG_SUCCESS} Password for '{username}' was changed.")


def handle_remove_staff():
    print_section_header("REMOVE STAFF ACCOUNT")
    username = pick_staff_account()
    if username is None:
        return
    confirm = input(f"  Remove '{username}'? (Y/N): ").strip().lower()
    if confirm == "y":
        remove_account(username)
        print(f"\n  {TAG_SUCCESS} Staff account '{username}' was removed.")
    else:
        print(f"\n  {TAG_CANCELLED} No changes were made.")


def handle_manage_accounts():
    while True:
        clear_screen()
        print_title("MANAGE STAFF ACCOUNTS")
        print("     [1] View Staff Accounts")
        print("     [2] Add Staff Account")
        print("     [3] Reset Staff Password")
        print("     [4] Remove Staff Account")
        print("     [5] Back to Main Menu")
        print("═" * LINE_WIDTH)
        choice = read_menu_choice("Select an option (1-5): ", 5)
        if choice == 5:
            return
        clear_screen()
        if choice == 1:
            display_staff_accounts()
        elif choice == 2:
            handle_add_staff()
        elif choice == 3:
            handle_reset_password()
        else:
            handle_remove_staff()
        pause("Press Enter to go back...")


def login():
    clear_screen()
    print_title("SCHOOL CANTEEN INVENTORY MONITORING SYSTEM",
                "Authorized Personnel Login")
    attempts = 0
    while attempts < MAX_LOGIN_ATTEMPTS:
        username = input("  Username: ").strip()
        password = input("  Password: ").strip()
        role = check_credentials(username, password)
        if role != "":
            print(f"\n  {TAG_SUCCESS} Welcome, {username} ({role})!")
            return [username, role]
        attempts += 1
        remaining = MAX_LOGIN_ATTEMPTS - attempts
        print(f"\n  {TAG_ERROR} Invalid username or password. "
              f"Attempts left: {remaining}\n")
    return []


def run_menu(username, role):
    options = get_menu_options(role)
    while True:
        clear_screen()
        display_menu(username, role, options)
        choice = read_menu_choice(f"Select an option (1-{len(options)}): ",
                                  len(options))
        action = options[choice - 1][0]
        clear_screen()
        if action == "view":
            display_all_items()
        elif action == "add":
            handle_add_item()
        elif action == "stock_in":
     
