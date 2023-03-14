# User Roles

1. **Patrons**: Users of the library.
2. **Staff**: Employees of the library who are responsible for managing the library's operations. Owns all permissions of Patrons.
3. **Administrators**: They can modify system settings and manage the account of patrons and staff. Could manage Staff identity. What's more, they can oversee the library's operations (health). Owns all permissions of Staff.
4. **Superusers**: Root users. Could manage Administrator identity. Owns all permissions of Administrators.
5. **Guests**: Could only search and view.

# Tips

>Entries with the same priority are supposed to implement together.

>**abstract material** and **material(copies / concrete material)**: 
>Book<u>s</u> called "Software Engineering" are <u>a</u> **abstract material**, which is *abstract*. <u>A</u> book called "Software Engineering" that arrived yesterday is a **copy** or **concrete material**, which is *concrete* and own a <u>call number</u> and is called <u>**material**</u> defaultly.

>If a material has any late fee, it can't be returned. Partons should pay online or pay to Staff who can clear late fees.  

>Column "Status" could be "To do", "In progress", or "Done".

# Product Backlog
| US ID | As a          | Want to                                                                                                      | Priority | Estimate | Status | Release |
|:----- |:------------- |:------------------------------------------------------------------------------------------------------------ |:-------- |:-------- |:------ | ------- |
| US02  | User          | have all permissions of lower-level users                                                                    | 1        | 1        | To do  | R01     |
| US01  | User          | login only by email without register                                                                         | 2        | 3        | To do  | R01     |
| GE01  | Guest         | enter main page without login                                                                                | 2        | 2        | To do  | R01     |
| AD02  | Administrator | view all accounts of Staff and Patron                                                                        | 3        | 1        | To do  | R01     |
| AD04  | Administrator | give or withdraw the Staff identity of an account                                                            | 3        | 1        | To do  | R01     |
| SU01  | Superusers    | give or withdraw the Administrator identity of an account                                                    | 3        | 1        | To do  | R01     |
| SF01  | Staff         | add new <u>abstract material</u> with metadata, and some metadata are automatically obtained through ISBN    | 4        | 8        | To do  | R01     |
| SF02  | Staff         | add new <u>material(copy / concrete material)</u> with concrete metadata, including call number and position | 4        | 4        | To do  | R01     |
| PT01  | Patron        | search for materials based on some keywords                                                                  | 6        | 5        | To do  | R01     |
| PT02  | Patron        | view all copies(concrete materials) of the selected abstract material                                        | 6        | 3        | To do  | R01     |
| SF04  | Staff         | modify tags(catalog) in search page                                                                          | 7        | 7        | To do  | R02     |
| SF03  | Staff         | modify metadata of materials and their copies                                                                | 8        | 4        | To do  | R02     |
| SF05  | Staff         | check out and in materials with call number                                                                  | 9        | 8        | To do  | R02     |
| SF06  | Staff         | get all call number of copies of a certain abstract material when checking out a book                        | 9        | 3        | To do  | R02     |
| AD06  | Administrator | modify system settings                                                                                       | 10       | 2        | To do  | R02     |
| PT04  | Patron        | view my borrowing history and due dates                                                                      | 11       | 3        | To do  | R03     |
| PT07  | Patron        | view how much late fees to be paid for every material borrowed                                               | 12       | 4        | To do  | R03     |
| PT06  | Patron        | receive an email reminding me to return the material when it is due                                          | 13       | 4        | To do  | R03     |
| PT05  | Patron        | renew items                                                                                                  | 14       | 2        | To do  | R03     |
| PT03  | Patron        | place a hold on a material                                                                                   | 15       | 5        | To do  | R03     |
| SF07  | Staff         | view lending history and due dates of every item                                                             | 16       | 3        | To do  | R03     |
| SF08  | Staff         | view how much late fees to be paid for a material                                                            | 16       | 1        | To do  | R03     |
| SF09  | Staff         | clear late fee of a material                                                                                 | 17       | 2        | To do  | R04     |
| PT08  | Patron        | pay for late fees                                                                                            | 18       | 5        | To do  | R04     |
| AD03  | Administrator | manage whether certain materials could be borrowed                                                           | 19       | 2        | To do  | R04     |
| AD05  | Administrator | ban or delete an account of staff and patrons                                                                | 20       | 1        | To do  | R04     |
| AD01  | Administrator | view the visual data of library operation                                                                    | 21       | 6        | To do  | R04     |

# User Story Mapping
## Parton
### Browse and reserve materials
- Search and browse
	- Search on type, title, author, topic or other keywords *R01*
	- Browse search results *R01*
	- Select a material *R01*
	- Select search tag *R02*
- Detail 
	- View materials' details *R01*
	- View copies' concrete details *R01*
- Reservation 
	- Reserve a material *R03*
	- Receive an email notification for availability *R03*
### Borrowing management
-  Handle due and renew
	- View borrowing history *R03*
	- View due dates *R03*
	- Select an item *R03*
	- Renew item *R03*
	- Receive an email notification when become due *R03*
- Late Fees
	- View late fees *R04*
	- Select an item *R04*
	- Pay for late fees *R04*

---
## Staff
### Material management
- Add new material
	- Add new abstract material *R01*
	- Add new material(copy / concrete material) *R01*
- Material message
	- Search by keywords *R01*
	- Browse search results *R01*
	- Select a material *R01*
	- Modify metadata *R02*
- Check in or out
	- Input ISBN *R02*
	- View all copies' call numbers *R02*
	- Select or input call numbers *R02*
	- Check in or out them *R02*
- Clear late fees
	- Select a material *R03*
	- View late fees *R03*
	- Clear late fees after offline payment *R04*
- Track materials
	- view lending history *R03*
	- view due dates *R04*
### Page management
- Manage search tags
	- input new tags *R02*
	- delete tags *R02*
	- modify tags' sort *R02*
	- modify tags' content *R02*
---
## Administrator
### Library's operations 
- Modify system settings
	- Modify maximum borrowing quantity *R02*
	- Modify longest borrowing time *R02*
	- Modify the extended days of each renew *R02*
	- Modify maximum renewing times *R02*
- Visual data
	- Select data type *R04*
	- View visual datas *R04*
- Borrowing permission
	- Select a material *R04*
	- Adjust borrowing permissions *R04*
### Account management
- Browse accounts
	- View all accounts of Partons or Staff *R01*
	- Search by identity or other keywords *R01*
	- Select an account *R01*
	- View details *R01*
- Manage Staff permission
	- Select an account of Partons or Staff *R01*
	- Give or withdraw the Staff identity *R01*
	- Ban or delete an account *R04*
---
## SuperUser
### Account management
- Manage Administrator permission
	- Select an account *R01*
	- Give or withdraw the Administrator identity of an account *R01*


# Sprint Backlog

| Sprint ID | US ID     | Task                                                               | Estimate(Hours) | Status |
| --------- |:--------- |:------------------------------------------------------------------ |:--------------- |:------ |
| SP001     | US02      | Design hierarchical permission logic for both frontend and backend | 3               | To do  |
| SP002     | US01      | Develop login functionality                                        | 4               | To do  |
| SP003     | US01      | Create login view                                                  | 5               | To do  |
| SP004     | GE01      | Add an access method without login                                 | 2               | To do  |
| SP005     | AD02      | Develop account message system                                     | 4               | To do  |
| SP006     | AD04,SU01 | Develop identity(permission) management system                     | 5               | To do  |
| SP007     | SF01      | Develop ISBN api                                                   | 4               | To do  |
| SP008     | SF02,SF01 | Develop material storing system                                    | 9               | To do  |
| SP009     | SF02,SF01 | Create adding view of materials                                    | 4               | To do  |
| SP010     | PT01      | Develop material-search functionality                              | 8               | To do  |
| SP011     | PT01      | Create material-search view                                        | 10              | To do  |
| SP012     | PT02      | Create material detail view, including all copies' message         | 5               | To do  |
| SP013     | SF04      | Develop tags management system                                     | 9               | To do  |
| SP014     | SF03      | Create metadata modifying view                                     | 4               | To do  |
| SP015     | SF05,SF06 | Develop check in and out functionality                             | 6               | To do  |
| SP016     | SF05,SF06 | Create check in and out view                                       | 8               | To do  |
| SP017     | AD06      | Develop system setting modifying system                            | 4               | To do  |
| SP018     | PT04      | Develop borrow management view                                     | 6               | To do  |
| SP019     | PT07,SF08 | Develop late fees calculation functionality                        | 3               | To do  |
| SP020     | PT06      | Develop due notification email system                              | 5               | To do  |
| SP021     | PT05      | Develop renew system                                               | 3               | To do  |
| SP022     | PT03      | Develop reserve system                                             | 5               | To do  |
| SP023     | PT03      | Develop reserve notification email system                          | 5               | To do  |
| SP024     | SF07      | Develop material track management functionality                    | 4               | To do  |
| SP025     | SF07      | Create material track management view                              | 6               | To do  |
| SP026     | SF09      | Add a function to clear late fee                                   | 2               | To do  |
| SP027     | PT08      | Develop late fees paying system                                    | 5               | To do  |
| SP028     | AD03      | Develop borrow permission system                                   | 4               | To do  |
| SP029     | AD05      | Add a function to ban or delete accounts                           | 3               | To do  |
| SP030     | AD01      | Develop visual data system                                         | 9               | To do  |


# Release Plan

| Release | Start Date | End Date  | Scope(Sprints) | Status |
| ------- | ---------- | --------- | -------------- | ------ |
| R01     | 2023/3/14  | 2023/4/4  | SP1~SP12       | To do  |
| R02     | 2023/4/5   | 2023/4/25 | SP13~SP17      | To do  |
| R03     | 2023/4/26  | 2023/5/16 | SP18~SP25      | To do  |
| R04     | 2023/5/17  | 2023/5/30 | SP26~SP30      | To do  |
































































