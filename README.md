# 📦 Purchase Order Request Management System

A low-code solution built with **Power Platform** tools—**Power Apps**, **SharePoint**, and **Power BI**—to streamline and automate the purchase order request process within an organization.

---

## 🧩 Project Overview

This system enables users to submit purchase requests through a Power Apps form, stores the data in SharePoint for centralized record-keeping, and generates insights through Power BI dashboards.

---

## 🎯 Objectives

- Automate submission and tracking of purchase orders.
- Reduce manual paperwork and errors.
- Enable real-time tracking and reporting.
- Improve transparency and accountability.

---

## 🔧 Tools & Technologies

| Tool            | Purpose                               |
|-----------------|----------------------------------------|
| Power Apps       | Front-end form for data entry         |
| SharePoint       | Backend data storage (SharePoint List)|
| Power BI         | Data visualization and reporting      |
| *(Optional)* Power Automate | Workflow automation & notifications |

---

## 🧱 System Architecture

1. User submits a request via Power Apps form.
2. Data is stored in the SharePoint list `Purchase Order Requests`.
3. Power BI is used for visualization and analytics.

---

## 🔐 Key Features

### ✅ Auto-Capture Submitter Details
Automatically captures the logged-in user's name and email using the `User()` function.

### 🔢 Unique PO Number Generation

```powerapps
Concatenate(
    "PO-",
    If(
        PayTDD.Selected.Value = "Item Purchase Order", "AP-",
        If(PayTDD.Selected.Value = "General Purchase Order", "GP-", "")
    ),
    Vartime
)
```

### 📋 Dynamic Item Collection
Users can enter multiple purchase items using a Power Apps form and store them in a local collection.

```powerapps
If(
    !IsBlank('txt-Amt_1'.Text) && 
    !IsBlank('txt-Iname_1'.Text) && 
    !IsBlank('txt-Qty_1'.Text) &&
    !IsBlank('txt-Itmdesc_1'.Text),
    Collect(
        colpurchase,
        {
            'Item Description': 'txt-Itmdesc_1'.Text,
            Item_Name: 'txt-Iname_1'.Text,
            Quantity: 'txt-Qty_1'.Text,
            Amount: 'txt-Amt_1'.Text
        }
    ),
    Notify("Add item: All fields are required.", NotificationType.Error)
);
Reset('txt-Amt_1');
Reset('txt-Iname_1');
Reset('txt-Qty_1');
Reset('txt-Itmdesc_1');
```

### 📤 Submission to SharePoint
Bulk submission of items in colpurchase to the SharePoint list:

```powerapps
ForAll(
    colpurchase,
    Patch(
        'Purchase Order Requests',
        Defaults('Purchase Order Requests'),
        {
            'Item Description': 'Item Description',
            Amount: Value(Amount),
            Quantity: Value(Quantity),
            'Item Name': Item_Name,
            'Payment Terms': Text(PayTDD.Selected.Value),
            'Due date': Duedate.SelectedDate,
            Institute: Text(InstiDD.Selected.Value),
            'PO Number': POID.Text,
            Approver: If(
                IsBlank(SelectApprover.Selected),
                Blank(),
                {
                    '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedUser",
                    Claims: "i:0#.f|membership|" & Lower(SelectApprover.Selected.Mail),
                    DisplayName: SelectApprover.Selected.DisplayName,
                    Department: "",
                    Email: SelectApprover.Selected.Mail,
                    JobTitle: ".",
                    Picture: "."
                }
            ),
            Requisitioner: {
                '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedUser",
                Claims: "i:0#.f|membership|" & Lower(User().Email),
                DisplayName: User().FullName,
                Department: "",
                Email: User().Email,
                JobTitle: ".",
                Picture: "."
            }
        }
    )
);
SubmitForm(POFORM);
ResetForm(POFORM);
Navigate(Thnaks);
```

### 📊 Power BI Report Features

- 📈 Visualize **total purchase value by category**
- 🏫 Monitor **number of POs per institute or approver**
- 📅 Apply filters by **date**, **requisitioner**, and **status**
- 📊 Track **monthly trends** and **top requested items**

---

### 🚀 Benefits

- ✔️ **Reduced manual entry** and paperwork
- ✔️ **Real-time visibility** of purchases
- ✔️ **Scalable and easy to maintain**
- ✔️ **Enhanced data integrity** and audit trail
