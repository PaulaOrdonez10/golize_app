# 👟 Golize – Power Apps, Powwer Automate, Share Point and Power Bi

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

![Image](https://github.com/user-attachments/assets/e1b1f314-8d30-4fdb-9319-0cfd7631b50d)

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

![Image](https://github.com/user-attachments/assets/8c533aa6-a75a-47fd-930d-76aa57ec7f76)

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

![Image](https://github.com/user-attachments/assets/317b2467-d070-44a2-a6f4-4ab940878bd0)

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

![Image](https://github.com/user-attachments/assets/8eef3e26-17f3-42f5-901b-1dd9bad0891c)

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

# ⚙️ Power Automate Integration

After placing an order, the customer receives a personalized email including a stylish HTML summary of their order.

### 🔁 Flow Steps

1. **Power Apps Trigger (V2)** – Collects `Orders`, `UserName`, and `UserEmail`
2. **Initialize variable** – Prepares a temporary variable if needed for flow logic
3. **Parse JSON** – Extracts the collection of ordered items using the schema below
4. **Create HTML Table** – Builds a customized table with formatted order details
5. **Compose HTML (Redactar)** – Adds inline CSS styles to the email
6. **Send Email (V2)** – Sends the final styled order summary to the user

---

### JSON Schema (Parse JSON)
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "Cantidad": { "type": "string" },
      "Category": { "type": "string" },
      "Customer_name": { "type": "string" },
      "Fecha": { "type": "string" },
      "Name": { "type": "string" },
      "Price": { "type": "string" },
      "Size": {
        "type": "object",
        "properties": {
          "Value": { "type": "string" }
        }
      },
      "Total": { "type": "integer" },
      "Type": { "type": "string" },
      "user": { "type": "string" }
    },
    "required": [
      "Cantidad", "Category", "Customer_name", "Fecha", "Name", "Price",
      "Size", "Total", "Type", "user"
    ]
  }
}
```

---

### 🖌️ HTML Table Styling Used
```html
<style>
table {
  border: 1px solid #1C6EA4;
  background-color: #EEEEEE;
  width: 100%;
  text-align: left;
  border-collapse: collapse;
}
table td, table th {
  border: 1px solid #AAAAAA;
  padding: 3px 2px;
}
table tbody td {
  font-size: 13px;
}
table thead {
  background: #1C6EA4;
  border-bottom: 2px solid #444444;
}
table thead th {
  font-size: 15px;
  font-weight: bold;
  color: #FFFFFF;
  border-left: 2px solid #D0E4F5;
}
table thead th:first-child {
  border-left: none;
}
</style>
```

---

# 📊 Power BI Dashboard

The SharePoint `ShoesOrders` list is connected directly to Power BI to generate a dynamic dashboard.

### Metrics Tracked:
- Product Category breakdown
- Type and Size distribution
- Count of products by Date
- Total Revenue by Date
- Best-Selling Products

![Image](https://github.com/user-attachments/assets/e7ba8b5e-c59b-4ea2-b96b-4edadab566d9)

---

Next up: How the dashboard refreshes, how to schedule it, and how to publish it to SharePoint or as a web report.



---

