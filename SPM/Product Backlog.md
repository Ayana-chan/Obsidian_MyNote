# User Roles

1. **Patrons**: Users of the library.
2. **Staff**: Employees of the library who are responsible for managing the library's operations. Owns all permissions of Patrons.
3. **Administrators**: They can modify system settings and manage the account of patrons and staff. Could manage Staff identity. What's more, they can oversee the library's operations (health). Owns all permissions of Staff.
4. **Superusers**: Root users. Could manage Administrator identity. Owns all permissions of Administrators.
5. **Guests**: Could only search and view.

# Product Backlog
| US ID | As a          | Want to                                                                                                                       | Priority | Estimate | Status |
|:----- |:------------- |:----------------------------------------------------------------------------------------------------------------------------- |:-------- |:-------- |:------ |
| US001 | User          | login only by email without register                                                                                          | 2        | 3        | To do  |
| US002 | User          | have all permissions of lower-level users                                                                                     | 1        | 1        | To do  |
| US003 | Guest         | enter main page without login                                                                                                 | 2        | 2        | To do  |
| US101 | Patron        | search for materials based on type, title, author, topic or other keywords                                                    | 6        | 5        | To do  |
| US102 | Patron        | view all copies(concrete materials) of the selected abstract material                                                         | 6        | 3        | To do  |
| US103 | Patron        | place a hold on a material so that I can receive an email notification when it's available and it won't be borrowed by others | 12       | 5        | To do  |
| US104 | Patron        | view my borrowing history and due dates                                                                                       | 10       | 3        | To do  |
| US105 | Patron        | renew items                                                                                                                   | 13       | 2        | To do  |
| US106 | Patron        | receive an email reminding me to return the material when it is due                                                           | 18       | 6        | To do  |
| US107 | Patron        | view how much late fees to be paid for every material borrowed                                                                | 11       | 4        | To do  |
| US108 | Patron        | pay for late fees                                                                                                             | 17       | 5        | To do  |
| US201 | Staff         | add new <u>abstract material</u> with metadata, and some metadata are automatically obtained through ISBN                     | 4        | 8        | To do  |
| US202 | Staff         | add new <u>copy(concrete material)</u> with concrete metadata, including call number and position                             | 4        | 4        | To do  |
| US203 | Staff         | modify metadata of materials and their copies                                                                                 | 8        | 4        | To do  |
| US204 | Staff         | modify tags(catalog) in search page                                                                                           | 7        | 7        | To do  |
| US206 | Staff         | get all call number of copies of a certain abstract material when checking out a book                                         | 9        | 3        | To do  |
| US205 | Staff         | check out and in materials with call number                                                                                   | 9        | 8        | To do  |
| US207 | Staff         | view lending history and due dates of every item                                                                              | 15       | 3        | To do  |
| US208 | Staff         | view how much late fees to be paid for a material                                                                             | 15       | 1        | To do  |
| US209 | Staff         | clear late fee of a material                                                                                                  | 16       | 2        | To do  | 
| US301 | Administrator | view the visual data of library operation                                                                                     | 21       | 6        | To do  |
| US302 | Administrator | view all accounts of Staff and Patron                                                                                         | 3        | 1        | To do  |
| US303 | Administrator | manage whether certain materials could be borrowed                                                                            | 19       | 2        | To do  |
| US304 | Administrator | give or withdraw the Staff identity of an account                                                                             | 3        | 1        | To do  |
| US305 | Administrator | ban or delete an account of staff and patrons                                                                                 | 20       | 1        | To do  |
| US401 | Superusers    | give or withdraw the Administrator identity of an account                                                                     | 3        | 1        | To do  |

# Tips

>**abstract material** and its **copies(concrete material)**: For example, book<u>s</u> called "Software Engineering" are <u>a</u> **abstract material**, which is *abstract*. <u>A</u> book called "Software Engineering" that arrived yesterday is a **copy** or **concrete material**, which is *concrete* and own a <u>call number</u>.

>If a material has any late fee, it can't be returned. Partons should pay online or pay to Staff who can clear late fees.  

>Column "Status" could be "To do", "In progress", or "Done".

# Questions



























