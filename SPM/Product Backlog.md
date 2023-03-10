# User Roles

1. **Patrons**: Users of the library.
2. **Staff**: Employees of the library who are responsible for managing the library's operations. Owns all permissions of Patrons.
3. **Administrators**: They can modify system settings and manage the account of patrons and staff. Could manage Staff identity. What's more, they can oversee the library's operations (health). Owns all permissions of Staff.
4. **Superusers**: Root users. Could manage Administrator identity. Owns all permissions of Administrators.
5. **Guests**: Could only search and view.

# Product Backlog
| US ID | As a          | Want to                                                                                                                       | Priority | Estimate | Status |
|:----- |:------------- |:----------------------------------------------------------------------------------------------------------------------------- |:-------- |:-------- |:------ |
| US001 | User          | login only by email without register                                                                                          | 2        | 2        | To do  |
| US002 | User          | have all permissions of lower-level users                                                                                     | 1        | 1        | To do  |
| US003 | Guest         | enter main page without login                                                                                                 | 2        | 1        | To do  |
| US011 | Patron        | view my borrowing history and due dates                                                                                       |          |          | To do  |
| US012 | Patron        | view how much late fees to be paid for every material borrowed                                                                |          |          | To do  |
| US013 | Patron        | view all copies(concrete materials) of the selected abstract material                                                         |          |          | To do  |
| US014 | Patron        | search for materials based on type, title, author, topic or other keywords                                                    | 3        | 5        | To do  |
| US015 | Patron        | renew items                                                                                                                   |          |          | To do  |
| US016 | Patron        | receive an email reminding me to return the material when it is due                                                           |          |          | To do  |
| US017 | Patron        | place a hold on a material so that I can receive an email notification when it's available and it won't be borrowed by others |          |          | To do  |
| US021 | Staff         | view lending history and due dates of every item                                                                              |          |          | To do  |
| US022 | Staff         | view how much late fees to be paid for the materials going to be checked in                                                   |          |          | To do  |
| US023 | Staff         | modify recommended classification(catalog) in search page                                                                     |          |          | To do  |
| US024 | Staff         | modify metadata of materials and their copies                                                                                 |          |          | To do  |
| US025 | Staff         | get all call number of copies of a certain abstract material when checking out a book                                         |          |          | To do  |
| US026 | Staff         | check out and in materials with call number                                                                                   |          |          | To do  |
| US027 | Staff         | add new <u>copy(concrete material)</u> with concrete metadata, including call number and position                             |          |          | To do  |
| US028 | Staff         | add new <u>abstract material</u> with metadata, and some metadata are automatically obtained through ISBN                     |          |          | To do  | 
| US031 | Administrator | view the visual data of library operation                                                                                     |          |          | To do  |
| US032 | Administrator | view all accounts of Staff and Patron                                                                                         |          |          | To do  |
| US033 | Administrator | manage whether certain materials could be borrowed                                                                            |          |          | To do  |
| US034 | Administrator | give or withdraw the Staff identity of an account                                                                             |          |          | To do  |
| US035 | Administrator | ban or delete an account of staff and patrons                                                                                 |          |          | To do  |
| US041 | Superusers    | give or withdraw the Administrator identity of an account                                                                     |          |          | To do  |

# Tips

>**abstract material** and its **copies(concrete material)**: For example, book<u>s</u> called "Software Engineering" are <u>a</u> **abstract material**, which is *abstract*. <u>A</u> book called "Software Engineering" that arrived yesterday is a **copy** or **concrete material**, which is *concrete* and own a <u>call number</u>.

>Column "Status" could be "To do", "In progress", or "Done".

# Questions



























