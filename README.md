# 👟 Golize – Power Apps App Overview (Part 1)

> *A modern sneaker ordering experience for young women, built using Microsoft Power Platform.*

---

## 🧹 Project Context

**Golize** is a digital sneaker ordering platform developed as a personal project to explore how low-code tools like Power Apps, SharePoint, and Power Automate can streamline and digitize retail operations. The goal is to offer a friendly, fast, and stylish customer experience while allowing business owners to manage inventory, receive orders, and track sales data.

---

## 🌟 Problem Statement

Small retail businesses often lack the technical resources to build fully automated ordering systems.  
**Golize** solves this by:

- Providing a simple and dynamic interface for customers (mainly young women) to browse and order sneakers.
- Automating the order processing and confirmation using flows.
- Tracking customer orders, product categories, types, and total revenue in a structured format.
- Visualizing business insights with Power BI (explained later).

---

## 🛠️ Technologies Used

| Tool             | Purpose                                                                 |
|------------------|-------------------------------------------------------------------------|
| **SharePoint**   | Two lists: one for sneaker inventory, another for customer orders       |
| **Power Apps**   | App with 4 screens for navigation, product selection, and order review  |
| **Power Automate** | Sends confirmation emails and registers order data                  |
| **Power BI**     | Will be used to visualize sales, categories, and customer data          |

---

## 🗃️ SharePoint Configuration

- **List 1** – `ShoesStore`: Stores the product catalog.
  - Fields: Title, Category, Type, Price, Image, Description
- **List 2** – `ShoesOrders`: Stores the submitted customer orders.
  - Fields: Title, Category, Type_, Price, Quantity, Images, Date (as Text), Customer, Total, Size_, User Name

Each field has the appropriate data type configured (text, number, image, etc.).

---

## 🧑‍💻 Power Apps Structure

The app contains **4 main screens**:

---

### 1️⃣ Welcome Screen

**Purpose**: To collect user's name and email before accessing the store.

**Components**:
- Inputs: `Input_name`, `Input_email`
- Logo Image
- Button with `OnSelect` logic:
```powerapps
If(
    "@" in Input_email.Text;
    Navigate('Main Menu'; ScreenTransition.Fade);
    Notify("Please enter a valid email with '@' to continue."; NotificationType.Error)
);;
Set(UserName; Input_name.Text);;
Set(UserEmail; Input_email.Text);;
```

---

### 2️⃣ Main Menu Screen

**Purpose**: To filter products by category and type.

**Components**:
- `Dropdown_category`: `Distinct(ShoesStore; Category)`
- `Dropdown_type`: `Distinct(Filter(ShoesStore; Category = Dropdown_category.Selected.Value); 'Type ')`
- `Gallery_ShoesStore`: Shows products using:
```powerapps
Filter(
    ShoesStore;
    Category = Dropdown_category.Selected.Value;
    'Type ' = Dropdown_type.Selected.Value
)
```

**Additional buttons**:
- **Back button**: Resets fields and returns to Welcome screen.
- **Reset filters**: Clears selected filters.

---

### 3️⃣ Detail Screen

**Purpose**: To show product details and let the user select size and quantity.

**Components**:
- `DisplayForm`: Fields from `ShoesStore` (Title, Price, Image, Description)
- `Dropdown_Size`: Items: `["S";"M";"L"]`
- `row_count`: Quantity input

**"Add to Cart" Button logic**:
```powerapps
Collect(
    Orders;
    {
        Name: Title_item.Text;
        Category: Category_item.Text;
        Type: Type_item.Text;
        Price: Price.Text;
        Size: Dropdown_Size.Selected;
        Cantidad: row_count.Text;
        Images: Image_screen_detail.Image;
        Total: Value(Price.Text) * Value(row_count.Text);
        Fecha: Text(Now(); "dd/mm/yyyy");
        user: User().FullName;
        Customer_name: UserName
    }
);;
Navigate('Main Menu'; ScreenTransition.Fade);;
```

**Back button**: `Back();`

---

### 4️⃣ Orders Screen

**Purpose**: Shows the customer’s order summary and allows submission.

**Components**:
- `Gallery_Orders`: Displays items from the `Orders` collection

**"Save Order" Button logic**:
```powerapps
ForAll(
    Orders;
    Patch(
        ShoesOrders;
        Defaults(ShoesOrders);
        {
            Title: Name;
            Category: Category;
            Type_: Type;
            Price: Value(Price);
            Quantity: Value(Cantidad);
            Images: Images;
            Date: Fecha;
            Customer: Customer_name;
            Total: Value(Total);
            Size_: Size.Value;
            user_name: UserEmail
        }
    )
);;
Set(
    varJSON;
    JSON(
        Orders;
        JSONFormat.IndentFour & JSONFormat.IgnoreBinaryData & JSONFormat.IgnoreUnsupportedTypes
    )
);;
sendJSON.Run(varJSON; UserName; UserEmail);;
Navigate(Welcome; ScreenTransition.Fade);;
Reset(Input_email);;
Reset(Input_name);;
Clear(Orders);;
```

**Extra Buttons**:
- Trash icon: Clears `Orders` and navigates back to Main Menu
- Back icon: `Back();`

---

Next steps will include how this data is sent using **Power Automate**, and visualized with **Power BI**.

---

