Software  use casess

6,7,8,13,14

A good way to make classes is put all necessary data in a table, then normalize it into different tables, then turn each table into a class/struct and then put all in a module and turn it into a database


Actors: modules, on stack dynamic dispatch, trait objects, strategy
1. What are the main tasks of each actor/role? 
2. Will the actor have to read/write/change any of the system information? 
3. Will the actor have to inform the system about outside changes? 
4. Does the actor wish to be informed about unexpected changes?



Interface: use traits and bridge pattern, genetics as type classes
1. Which present information to the actor or request information from him, 
2. The functionality of which is changed if the actor's behavior is changed, 
3. Where a course is dependent on a particular interface type.


Entity - use new type often to encapsulate, type state
The following is a list of typical operations that must be offered by an entity object: 
1. Storing and fetching information, 
2. Courses that must be changed if the entity object is changed, 
3.  Creating and removing the entity object.


Controllers:generic as typeclasses, fold pattern
1. Will changes of one object lead to changes in the other object? 
2. Do they communicate with the same actor? 
3. Are both of them dependent on a third object, such as an interface object or an entity object? 
4. Does one object perform several operations on the other?






Components
A more active way of finding components has been by Caldiera and Basili (1991). Various metrics of propose a what is typical fc reusable component are used to identify components. A program that calculates these measures for existing source code has been developed and produces candidates for components. Hundreds thousands of lines of C source code have then been the result looks promising. The metrics used include:
* Size This affects both reuse cost the and quality. If it is too small benefits will not exceed the cost of managing it. If it is toco large, it is hard to have high quality. 
* Complexity. This affects also reuse cost and quality. A too trivial component is not profitable to reuse and with a too complex component it is hard to have high quality. 
* Reuse frequency. The number of places where a component used is of course important too.



* Reduce the number of parameters; fewer will lead to more cohesive operations that also imply more primitive operations, 
* Avoid using options in parameters; any options should be set by the creation procedure, and special operations should be used to change these options, 
* No direct access to instance variables; to access instance variables, special operations should be used, this frees the implementation from the actual interface, 
* Naming should be consistent; operations in different com- similar names if they perform the same ponents should have task



Use composition first, then new type, then traits, then inheritance to use components 
  <img width="755" alt="Structures" src="https://github.com/MustCodeAl/programming_mental_notes/assets/87888006/428f39e8-d125-46b2-ad04-d92f5da5eb24">
￼
  structures
To handle changes of how a sequence is actually performed, you should encapsulate the actual sequence and thus obtain decentralized structure. If you want to change the actual order of the operations, it is better to have a centralized stucture, since then such changes will be local to one object. We thus see that both structures have their benefits What is crucial is whether the operations have strong connection to each objects: other. A strong connection exists if the objects

1. Form a 'consists-of' hierarchy, such as country-state-city,
2.  form an information hierarchy, such as document-chapter- section-paragraph-character, 
3. represent a fixed temporal relationship, such as advertisement- order-invoice-delivery-payment, 
4. form (conceptual) inheritance mammal-cat hierarchy, such as animal-

 We have found the following principal rules: . 
A decentralized (stair) control structure is appropriate when: 
1. The operations have a strong connection (see above), 
2. The operations will always be performed in the same order, or 
3. You want to abstract or encapsulate behavior. 

A centralized (fork) control structure is appropriate when: 
1. The operations can change order, 
2. New operations could be inserted
