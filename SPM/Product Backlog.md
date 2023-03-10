# User Roles

1. **Patrons**: Users of the library.
2. **Staff**: Employees of the library who are responsible for managing the library's operations. Owns all permissions of Patrons.
3. **Administrators**: They can modify system settings and manage the account of patrons and staff. Could give account staff identity. What's more, they can oversee the library's operations (health). Owns all permissions of Staff.
4. **Superusers**: Root users. Could give account administrator identity. Owns all permissions of Administrators.
5. **Guests**: Could only search and view.

# Product Backlog
| US ID | As a                                     | Want to                                                                                                                                  | Priority | Estimate | Status |
|:----- |:---------------------------------------- |:---------------------------------------------------------------------------------------------------------------------------------------- |:-------- |:-------- |:------ |
|       | User                                     | have all permissions of low-level users                                                                                                  | 1        | 1        | To do  |
| US001 | Patron, Staff, Administrator, Superusers | login only by email without register                                                                                                     | 2        | 2        | To do  |
| US002 | Guest                                    | enter main page without login                                                                                                            | 2        | 1        | To do  |
| US003 | Patron                                   | search for materials based on type, title, author, topic or other keywords                                                               | 3        | 5        | To do  |
|       | Patron                                   | view the detail of a material selected, and show me all copies(concrete materials) of a certain abstract material                        |          |          |        |
|       | Patron                                   | place a hold on a material so that I can receive an email notification when it's available and it won't be lent to others                |          |          |        |
|       | Patron                                   | view my borrowing history and due dates                                                                                                  |          |          |        |
|       | Patron                                   | receive an email reminding me to return the material when it is due                                                                      |          |          |        |
|       | Patron                                   | view how much late fees to be paid for every material borrowed                                                                           |          |          |        |
|       | Patron                                   | renew items                                                                                                                              |          |          |        |
|       | Staff                                    | add new <u>abstract material</u> with metadata, and some metadata are automatically obtained through ISBN                                |          |          |        |
|       | Staff                                    | add new <u>copy(concrete material)</u> with concrete metadata, including call number and position, determining attribution based on ISBN |          |          |        |
|       | Staff                                    | modify metadata of materials and their copies                                                                                            |          |          |        |
|       | Staff                                    | check out and in materials with call number                                                                                              |          |          |        |
|       | Staff                                    | get all call number of copies of a certain abstract material through ISBN when checking out a book                                       |          |          |        |
|       | Staff                                    | view how much late fees to be paid for the materials going to be checked in                                                              |          |          |        |
|       | Staff                                    | view lending history and due dates of every item                                                                                         |          |          |        |
|       | Staff                                    | modify recommended classification(catalog) in search page                                                                                |          |          |        |
|       | Administrator                            | view all accounts of Staff and Patron                                                                                                    |          |          |        |
|       | Administrator                            | ban or delete an account of staff and patrons                                                                                            |          |          |        |
|       | Administrator                            | give or withdraw the Staff identity of an account                                                                                        |          |          |        |
|       | Administrator                            | view the visual data of library operation                                                                                                |          |          |        |
|       | Administrator                            | manage whether certain materials could be borrowed                                                                                       |          |          |        |
|       | Superusers                               | give or withdraw the Administrator identity of an account                                                                                                     |          |          |        |

# Tips

>**abstract material** and its **copies(concrete material)**: For example, book<u>s</u> called "Software Engineering" are <u>a</u> **abstract material**, which is *abstract*. <u>A</u> book called "Software Engineering" that arrived yesterday is a **copy** or **concrete material**, which is *concrete* and own a <u>call number</u>.

>Column "Status" could be "To do", "In progress", or "Done".

# Questions



























