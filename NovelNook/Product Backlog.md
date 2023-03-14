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
| PT03  | Patron        | place a hold on a material                                                                                   | 13       | 5        | To do  | R03     |
| PT05  | Patron        | renew items                                                                                                  | 14       | 2        | To do  | R03     |
| SF07  | Staff         | view lending history and due dates of every item                                                             | 15       | 3        | To do  | R03     |
| SF08  | Staff         | view how much late fees to be paid for a material                                                            | 15       | 1        | To do  | R03     |
| SF09  | Staff         | clear late fee of a material                                                                                 | 16       | 2        | To do  | R03     |
| PT08  | Patron        | pay for late fees                                                                                            | 17       | 5        | To do  | R03     |
| PT06  | Patron        | receive an email reminding me to return the material when it is due                                          | 18       | 6        | To do  | R04     |
| AD03  | Administrator | manage whether certain materials could be borrowed                                                           | 19       | 2        | To do  | R04     |
| AD05  | Administrator | ban or delete an account of staff and patrons                                                                | 20       | 1        | To do  | R04     |
| AD01  | Administrator | view the visual data of library operation                                                                    | 21       | 6        | To do  | R04     |





























