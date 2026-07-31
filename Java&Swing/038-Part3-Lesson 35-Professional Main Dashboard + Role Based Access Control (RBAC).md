# Part 3: Café POS Real Implementation Phase

# Lesson 35: Professional Main Dashboard + Role Based Access Control (RBAC)

## Main Window Architecture + Sidebar Navigation + Permission System

### (Java 25 + Swing + MVC + FlatLaf + Security Architecture)

ဒီ Lesson မှာ Café POS ရဲ့ **Main Application Shell** ကို တည်ဆောက်ပါမယ်။

အခုအချိန်မှာ:

```text
Application Start

        ↓

Login Screen

        ↓

Authentication Success

        ↓

Session Created

```

အထိ ရပါပြီ။

ဒီနေ့ပြီးရင်:

```text
Login

   ↓

Main Dashboard

   ↓

Navigation Menu

   ↓

Permission Check

   ↓

Open Modules

```

ဖြစ်လာပါမယ်။

---

# 1. Main Dashboard Architecture

Professional Desktop App:

```text
+------------------------------------------------+

| Café POS                 User: Admin  Logout   |

+------------------------------------------------+

|          |                                   |

| Sidebar  |                                   |

|          |          Content Panel            |

| Dashboard|                                   |

| Product  |                                   |

| Order    |                                   |

| Inventory|                                   |

| Report   |                                   |

|          |                                   |

+----------+-----------------------------------+

```

---

# 2. Package Structure

Create:

```text
view

├── MainFrame.java

├── DashboardPanel.java

├── components

│      ├── SidebarPanel.java

│      ├── HeaderPanel.java

│      └── MenuButton.java

```

---

# 3. MainFrame Responsibility

MainFrame should only manage:

```text
Window

Layout

Panel Switching

User Session Display

```

---

It should NOT contain:

❌ Database code

❌ Business logic

❌ Permission logic

---

# 4. Application Flow

Current:

```text
Application.java


        |

        |

LoginFrame


        |

        |

Login Success


        |

        |

MainFrame


```

---

# 5. Update Login Success

Before:

```java
User user =
controller.login(
username,
password
);

```

After:

```java
User user =
controller.login(
username,
password
);


new MainFrame()
.setVisible(true);


dispose();

```

---

# 6. Create MainFrame

File:

```text
view/MainFrame.java

```

---

Code:

```java
package com.cafe.pos.view;


import javax.swing.*;

import java.awt.*;


public class MainFrame
extends JFrame {


private JPanel contentPanel;



public MainFrame(){


setTitle(
"Café POS System"
);


setSize(
1400,
800
);


setLocationRelativeTo(null);


setDefaultCloseOperation(
EXIT_ON_CLOSE
);



initialize();


}



private void initialize(){


setLayout(
new BorderLayout()
);



add(
new HeaderPanel(),

BorderLayout.NORTH
);



add(
new SidebarPanel(
this
),

BorderLayout.WEST
);



contentPanel =
new JPanel(
new BorderLayout()
);



add(
contentPanel,

BorderLayout.CENTER
);



showDashboard();


}



public void changeContent(
JPanel panel
){


contentPanel.removeAll();


contentPanel.add(
panel,
BorderLayout.CENTER
);


contentPanel.revalidate();


contentPanel.repaint();


}



private void showDashboard(){


changeContent(
new DashboardPanel()
);


}


}

```

---

# 7. Header Panel

Purpose:

```text
Current User

Role

Logout

```

---

Create:

```text
components/HeaderPanel.java

```

---

Code:

```java
public class HeaderPanel
extends JPanel{


public HeaderPanel(){


setLayout(
new FlowLayout(
FlowLayout.RIGHT
)
);


JLabel user =
new JLabel(
"User: Admin"
);


JButton logout =
new JButton(
"Logout"
);



add(user);

add(logout);


}


}

```

---

# 8. Sidebar Navigation

Menu:

```text
Dashboard

Products

Inventory

Orders

Reports

Users

Logout

```

---

Create:

```text
components/SidebarPanel.java

```

---

Code:

```java
public class SidebarPanel
extends JPanel{


private MainFrame mainFrame;



public SidebarPanel(
MainFrame mainFrame
){


this.mainFrame =
mainFrame;


setLayout(
new GridLayout(
10,
1
)
);



addButton(
"Dashboard"
);



addButton(
"Products"
);



addButton(
"Inventory"
);



addButton(
"Orders"
);



addButton(
"Reports"
);



}



private void addButton(
String name
){


JButton button =
new JButton(name);


add(button);


}


}

```

---

# 9. Dashboard Panel

Create:

```text
DashboardPanel.java

```

---

Code:

```java
public class DashboardPanel
extends JPanel{


public DashboardPanel(){


JLabel label =
new JLabel(
"Welcome Dashboard"
);



label.setHorizontalAlignment(
SwingConstants.CENTER
);



add(label);


}


}

```

---

# 10. Why Content Switching?

Bad:

```java
new ProductFrame();

new OrderFrame();

new ReportFrame();

```

Problem:

Many windows.

---

Professional:

```text
One MainFrame

      |

Change JPanel

```

---

Like:

```text
MainFrame


+----------------+

| Sidebar        |

|                |

| Content Panel  |

|                |

+----------------+

```

---

# 11. Role Based Access Control (RBAC)

Now security.

Example:

ADMIN:

```text
Dashboard

Products

Inventory

Orders

Reports

Users

```

---

CASHIER:

```text
Dashboard

Orders

Payment

```

---

MANAGER:

```text
Dashboard

Inventory

Reports

Orders

```

---

# 12. Permission Design

Create:

```text
security

└── Permission.java

```

---

Enum:

```java
public enum Permission {


PRODUCT_CREATE,

PRODUCT_DELETE,

PRODUCT_VIEW,


ORDER_CREATE,

ORDER_CANCEL,


INVENTORY_ADJUST,


REPORT_VIEW,


USER_MANAGE


}

```

---

# 13. Permission Table Design

Database:

```sql
CREATE TABLE permissions(

id BIGINT PRIMARY KEY AUTO_INCREMENT,

name VARCHAR(100) UNIQUE

);

```

---

Role Permission:

```sql
CREATE TABLE role_permissions(

role_id BIGINT,

permission_id BIGINT,


FOREIGN KEY(role_id)

REFERENCES roles(id),


FOREIGN KEY(permission_id)

REFERENCES permissions(id)

);

```

---

# 14. Permission Service

Create:

```text
security

PermissionService.java

```

---

Code:

```java
public class PermissionService {



public boolean hasPermission(
Permission permission
){


User user =
Session.getUser();



if(user==null)

return false;



String role =
user.role()
.name();



if(role.equals("ADMIN"))

return true;



return false;


}


}

```

---

# 15. Hide Menu Based on Role

Example:

Sidebar:

```java
if(
permissionService
.hasPermission(
Permission.USER_MANAGE
)
){


add(usersButton);


}

```

---

Result:

Admin:

```text
Users Menu

Visible

```

---

Cashier:

```text
Users Menu

Hidden

```

---

# 16. Session User Display

Header:

Before:

```java
"User: Admin"

```

Better:

```java
User user =
Session.getUser();


String text =
user.username()
+
"("
+
user.role().name()
+
")";

```

---

Display:

```text
admin (ADMIN)

```

---

# 17. Logout Function

Flow:

```text
Click Logout


    |

Session.clear()


    |

Close MainFrame


    |

Open LoginFrame


```

---

Code:

```java
logoutButton.addActionListener(e->{


Session.clear();


dispose();


new LoginFrame()
.setVisible(true);



});

```

---

# 18. FlatLaf Professional Theme

Application:

```java
FlatDarkLaf.setup();

```

or:

```java
FlatLightLaf.setup();

```

---

Professional POS:

Dark:

```text
Black Header

Gray Sidebar

White Content

```

---

# 19. Dashboard Architecture Now

```text
              MainFrame


                  |

        --------------------

        |                  |

   HeaderPanel        SidebarPanel


                  |

             ContentPanel


                  |

        -----------------

        |       |       |

   Product  Order  Report


```

---

# 20. Current Project Status

Completed:

```text
Project Setup              ✅

Database Layer             ✅

Migration System            ✅

Authentication              ✅

BCrypt Security             ✅

Session Management           ✅

Main Dashboard               ✅

Sidebar Navigation           ✅

RBAC Foundation              ✅

```

---

# Practice Task

Implement:

1. Create MainFrame
    
2. Create HeaderPanel
    
3. Create SidebarPanel
    
4. Create DashboardPanel
    
5. Connect Login → MainFrame
    
6. Add Session Display
    
7. Add Logout
    

---

# Next Lesson

# Lesson 36: Product Management Module (Part 1)

## Entity + Repository + Service + CRUD Backend

Next we start the first real business module:

```text
Product

Database

↓

Entity

↓

Repository

↓

Service

↓

Controller

↓

Swing JTable UI

```

ပြီးရင် Café POS မှာ **Product Add/Edit/Delete/Search တကယ်အလုပ်လုပ်ပါမယ်။**