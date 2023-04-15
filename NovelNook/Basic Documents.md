# Project Name
**NovelNook A2**
# User Roles

1. **Patrons**: Users of the library.
2. **Staff**: Employees of the library who are responsible for managing the library's operations. Owns all permissions of Patrons.
3. **Administrators**: They can modify system settings and manage the account of patrons and staff. Could manage Staff permission. What's more, they can oversee the library's operations (health). Owns all permissions of Staff.
4. **Superusers**: Root users. Could manage Administrator permission.
5. **Guests**: Could only search and view.

# Sub-Teams
### Partons *7*
- 石佳欣 *Scrum Master*
- 朱玲玲
- 刘潇驰
- 陈伊婧
- 莫忠仁
- 岳浩东
- 王立文
### Staff *8*
- 魏英豪 *Scrum Master*
- 刘君
- 王馨楠
- 李正言
- 高翔宇
- 刘海晨
- 史卓一
- 张一栋
### Administrators *6*
- 魏柏斌 *Scrum Master*
- 夏雨烟
- 赵云骁
- 陈秋环
- 吴泉霖
- 李颖
### Superusers & Guests *4*
- 强晓东 *Scrum Master*
- 李爽
- 陈路路
- 夏奥

# Tips

>Entries with the same priority are supposed to implement together.

>**abstract material** and **material(copies / concrete material)**: 
>Book<u>s</u> called "Software Engineering" are <u>a</u> **abstract material**, which is *abstract*. 
><u>A</u> book called "Software Engineering" that arrived yesterday is a **copy** or **concrete material**, which is *concrete* and own a <u>call number</u> and is called <u>**material**</u> defaultly.

>If a material has any late fee, it can't be returned. Partons should pay online or pay to Staff who can clear late fees.  

>Column "Status" could be "To do", "In progress", or "Done".

# Product Backlog
| US ID    | As a          | Want to                                                                                                      | Priority | Estimate | Status | Release |
|:-------- |:------------- |:------------------------------------------------------------------------------------------------------------ |:-------- |:-------- |:------ | ------- |
| US01     | User          | login only by email without register                                                                         | 2        | 3        | To do  | R01     |
| GE01     | Guest         | enter main page without login                                                                                | 2        | 2        | To do  | R01     |
| AD02     | Administrator | view all accounts of Staff and Patron                                                                        | 3        | 1        | To do  | R01     |
| AD04     | Administrator | give or withdraw the Staff permission of an account                                                          | 3        | 1        | To do  | R01     |
| SU01     | Superuser     | give or withdraw the Administrator permission of an account                                                  | 3        | 1        | To do  | R01     |
| SF01     | Staff         | add new <u>abstract material</u> with metadata, and some metadata are automatically obtained through ISBN    | 4        | 8        | To do  | R01     |
| SF02     | Staff         | add new <u>material(copy / concrete material)</u> with concrete metadata, including call number and position | 4        | 4        | To do  | R01     |
| PT01     | Patron        | search for materials based on some keywords                                                                  | 6        | 5        | To do  | R01     |
| PT02     | Patron        | view all copies(concrete materials) of the selected abstract material                                        | 6        | 3        | To do  | R01     |
| AD06     | Administrator | modify tags(catalog) in search page                                                                          | 7        | 7        | To do  | R02     |
| SF03     | Staff         | modify metadata of materials and their copies                                                                | 8        | 4        | To do  | R02     |
| SF05     | Staff         | check out and in materials with call number                                                                  | 9        | 8        | To do  | R02     |
| SF06     | Staff         | get all call number of copies of a certain abstract material when checking out a book                        | 9        | 3        | To do  | R02     |
| AD06     | Administrator | modify system settings                                                                                       | 10       | 2        | To do  | R02     |
| PT04     | Patron        | view my borrowing history and due dates                                                                      | 11       | 3        | To do  | R03     |
| PT07     | Patron        | view how much late fees to be paid for every material borrowed                                               | 12       | 4        | To do  | R03     |
| PT06     | Patron        | receive an email reminding me to return the material when it is due                                          | 13       | 4        | To do  | R03     |
| ~~PT05~~ | Patron        | renew items                                                                                                  | 14       | 2        | To do  | R03     |
| PT03     | Patron        | place a hold on a material                                                                                   | 15       | 5        | To do  | R03     |
| SF07     | Staff         | view lending history and due dates of every item                                                             | 16       | 3        | To do  | R03     |
| SF08     | Staff         | view how much late fees to be paid for a material                                                            | 16       | 1        | To do  | R03     |
| SF09     | Staff         | clear late fee of a material                                                                                 | 17       | 2        | To do  | R04     |
| PT08     | Patron        | pay for late fees                                                                                            | 18       | 5        | To do  | R04     |
| AD03     | Administrator | manage whether certain materials could be borrowed                                                           | 19       | 2        | To do  | R04     |
| AD05     | Administrator | ban or delete an account of staff and patrons                                                                | 20       | 1        | To do  | R04     |
| AD01     | Administrator | view the visual data of library operation                                                                    | 21       | 6        | To do  | R04     |

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
	- ~~Renew item~~ *R03*
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
	- Give or withdraw the Staff permission *R01*
	- Ban or delete an account *R04*

### Page management
- Manage search tags
	- input new tags *R02*
	- delete tags *R02*
	- modify tags' sort *R02*
	- modify tags' content *R02*
---
## SuperUser
### Account management
- Give Administrator permission
	- Input an account's ID(email) *R01*
	- Give the Administrator permission *R01*
- Withdraw Administrator permission
	- View Administrators' accounts *R01*
	- Select an account *R01*
	- Withdraw the Administrator permission *R01*


# Sprint Backlog

| Sprint ID | US ID     | Task                                                       | Estimate(Hours) | Status | Release |
| --------- |:--------- |:---------------------------------------------------------- |:--------------- |:------ | ------- |
| SP001     | US01,GE01 | Develop login system                                       | 4               | To do  | R01     |
| SP002     | AD02      | Develop account message system                             | 4               | To do  | R01     |
| SP003     | AD04,SU01 | Develop permission management system                       | 7               | To do  | R01     |
| SP004     | SF02,SF01 | Develop material storing system                            | 9               | To do  | R01     |
| SP005     | PT01      | Develop material-search functionality                      | 8               | To do  | R01     |
| SP006     | PT02      | Create material detail page, including all copies' message | 5               | To do  | R01     |
| SP007     | AD06      | Develop tags management system                             | 9               | To do  | R02     |
| SP008     | SF03      | Create metadata modifying page                             | 4               | To do  | R02     |
| SP009     | SF05,SF06 | Develop check in and out system                            | 6               | To do  | R02     |
| SP010     | PT04      | Develop borrow management page                             | 6               | To do  | R03     |
| SP011     | PT07,SF08 | Develop late fees calculation functionality                | 3               | To do  | R03     |
| SP012     | PT06      | Develop due notification email system                      | 5               | To do  | R03     |
| SP013     | PT03      | Develop reserve system with notification email system      | 5               | To do  | R03     |
| SP014     | SF07      | Develop material track management functionality            | 4               | To do  | R03     |
| SP015     | SF09      | Add a function to clear late fee                           | 2               | To do  | R04     |
| SP016     | PT08      | Develop late fees paying system                            | 5               | To do  | R04     |
| SP017     | AD03      | Develop borrow permission system                           | 4               | To do  | R04     |
| SP018     | AD05      | Add a function to ban or delete accounts                   | 3               | To do  | R04     |
| SP019     | AD01      | Develop visual data system                                 | 9               | To do  | R04     |


# Release Plan

| Release | Start Date | End Date  | Status |
| ------- | ---------- | --------- | ------ |
| R01     | 2023/3/21  | 2023/4/11 | To do  |
| R02     | 2023/4/12  | 2023/5/2  | To do  |
| R03     | 2023/5/3   | 2023/5/30 | To do  |

































































