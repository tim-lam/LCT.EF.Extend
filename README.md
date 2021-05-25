# LCT.EntityFramework.GenericRepository

LCT.EntityFramwork.GenericRepository extends EF features and eases C R U of "CRUD" operations. With a single generic repository, it can:
 
  - add or update your domain model with single method.
  - retreive data and map to a different defined model or an Anonymous type.

It is available https://www.nuget.org/packages/LCT.EntityFramework.Extensions.

## Concepts

Any domain and it's related models in the object graph will be added or updated which compares existing model from databse.  _AddOrUpdate_ is a single method that handles add or update.  It starts the model passing into the method and recursively does same comparision and operation for each model in object graph.

Deletion will operate the same way as default behavior of Entity Framework.
It follows the same practice from SQL database.  **Cascade Delete** can be configured with Fluent Configuration; 
please see details: https://docs.microsoft.com/en-us/ef/ef6/modeling/code-first/fluent/relationships

There are several retrieval methods that will retrieve a single object or a collection of defined models. These retrieval methods will also allow to map to anonymous type; meaning these methods will be able to handle most cases without implementing new specific repository for a specific model.

## How do use
LCT.EntityFramework.GenericRepository can be used and replace DBContext.  It will handle add or update with a single method, AddOrUpdate 
as long as Entity Models are configured properly with Fluent Configuration.

Assuming there is a _SchoolDbContext_, _Teacher_ is an Entity Model and it has _Classes_ which is a collection of _Class_ Entity Model. 
Using AddOrUpdate can:
- Add a new Teacher
- Update an existing Teacher
- Add new _Class_ of _Classes_ collection while AddOrUpdate the Teacher object
- Update an existing _Class_ of _Classes_ collection

Example:
>      var teacher = new Teacher {
>                    Classes = new List<Class> {
>                              new Class { Name = "New Class" } 
>                              new Class { Id = 1, "Existing Class" } //will update if any properties changed.
>                              }
>                    };
>      var teacherRepository = new GenericRepository<Teacher>(SchoolDbContext);
>      teacherRepository.AddOrUpdate(teacher);

Two main different types of operation that align with CQRS: Command and Quary.
### Command
Command is mainly for _Add_, _Update_, _Delete_ and extra extended _AddOrUpdate_. 
_Add_, _Update_ and _Delete_ are same _DbContext_ operations as expected and documented for Entity Framework in MSDN. 


**_AddOrUpdate_** method is main concerpt and reason that this project has created.  _AddOrUpdate_ is trying to address challenge that a single object with navigation properties and it requires some rules to determine add operation or update operation for single object or a collection of object for different navigation properties.  It will handle:
- Determine Add operation or update operation for an object and all associated objects or collections recursively.
- Add new object recursively to database if no existing object found.
- Update changed properties only of existing object.
- Delete object if it is not in collection navigation property.

_AddOrUpdate_ method handles the challenge with single method and single repository and creates cleaner and simplier data access layer.

Query is mainly for data retrieval. 

### Standard (_DbContext_) Command Methods
- _void Add(T entity);_
- _void Update(T entity);_
- _void Delete(T entity);_
- _T Single(Expression<Func<T, bool>> predicate);_
- _Task<T> SingleAsync(Expression<Func<T, bool>> predicate);_
- _T SingleOrDefault(Expression<Func<T, bool>> predicate);_
- _Task<T> SingleOrDefaultAsync(Expression<Func<T, bool>> predicate);_
- _IEnumerable<T> ToList();_
- _Task<IEnumerable<T>> ToListAsync();_

### Extended Command Methods
- _void AddOrUpdate(T entity);_
  - Operate add or update for object itself and it's object graph that based on any property changes.  

- _Task AddOrUpdateAsync(T entity);_
  - Operate async with same expectation as described for AddOrUpdate. 

### Extended Query Methods
- _T Single(
    Expression<Func<T, bool>> predicate, 
    params Expression<Func<T, dynamic>>[] paths);_
  - _predicate_ is filter and used along with _Where_.
  - _paths_ specifies the related objects in query results and is used along with _Include_.

- _Task<T> SingleAsync(
    Expression<Func<T, bool>> predicate,
    params Expression<Func<T, dynamic>>[] paths);_
  - _predicate_ is filter and used along with _Where_.
  - _paths_ specifies the related objects in query results and is used along with _Include_.
  - operates asynchronously.
 
- _T SingleOrDefault(
    Expression<Func<T, bool>> predicate,
    params Expression<Func<T, dynamic>>[] paths);_
  - _predicate_ is filter and used along with _Where_.
  - _paths_ specifies the related objects in query results and is used along with _Include_.

- _Task<T> SingleOrDefaultAsync(
    Expression<Func<T, bool>> predicate,
    params Expression<Func<T, dynamic>>[] paths);_
  - _predicate_ is filter and used along with _Where_.
  - _paths_ specifies the related objects in query results and is used along with _Include_.
  - operates asynchronously.

- _TResult SingleOf<TResult>(
    Expression<Func<T, TResult>> selector,
    Expression<Func<T, bool>> predicate);_
  - _selector_ projects each element of a sequence into a new form.
  - _predicate_ is filter and used along with _Where_.

- _Task<TResult> SingleOfAsync<TResult>(
    Expression<Func<T, TResult>> selector,
    Expression<Func<T, bool>> predicate);_
  - _selector_ projects each element of a sequence into a new form.
  - _predicate_ is filter and used along with _Where_.
  - operates asynchronously.

- _TResult SingleOrDefaultOf<TResult>(
    Expression<Func<T, TResult>> selector,
    Expression<Func<T, bool>> predicate);_
  - _selector_ projects each element of a sequence into a new form.
  - _predicate_ is filter and used along with _Where_.


- _Task<TResult> SingleOrDefaultOfAsync<TResult>(
    Expression<Func<T, TResult>> selector,
    Expression<Func<T, bool>> predicate);_
  - _selector_ projects each element of a sequence into a new form.
  - _predicate_ is filter and used along with _Where_.
  - operates asynchronously.

- _IEnumerable<T> ToList(Expression<Func<T, bool>> predicate);_
  - _predicate_ is filter and used along with _Where_.

- _IEnumerable<T> ToList(params Expression<Func<T, dynamic>>[] paths);_
  - _paths_ specifies the related objects in query results and is used along with _Include_.

- _IEnumerable<T> ToList(
    Expression<Func<T, bool>> predicate,
    params Expression<Func<T, dynamic>>[] paths);_
  - _predicate_ is filter and used along with _Where_.
  - _paths_ specifies the related objects in query results and is used along with _Include_.
  
- _Task<IEnumerable<T>> ToListAsync(Expression<Func<T, bool>> predicate);_
  - _predicate_ is filter and used along with _Where_.

- _Task<IEnumerable<T>> ToListAsync(params Expression<Func<T, dynamic>>[] paths);_
  - _paths_ specifies the related objects in query results and is used along with _Include_.
  
- _Task<IEnumerable<T>> ToListAsync(
    Expression<Func<T, bool>> predicate,
    params Expression<Func<T, dynamic>>[] paths);_
  - _predicate_ is filter and used along with _Where_.
  - _paths_ specifies the related objects in query results and is used along with _Include_.
  - operates asynchronously.

- _IEnumerable<TResult> ToListOf<TResult>(Expression<Func<T, TResult>> selector);_
  - _selector_ projects each element of a sequence into a new form.
  
- _IEnumerable<TResult> ToListOf<TResult>(
    Expression<Func<T, TResult>> selector,
    Expression<Func<T, bool>> predicate);_
  - _selector_ projects each element of a sequence into a new form.
  - _predicate_ is filter and used along with _Where_.

- _Task<IEnumerable<TResult>> ToListOfAsync<TResult>(Expression<Func<T, TResult>> selector);_
  - _selector_ projects each element of a sequence into a new form.
  - operates asynchronously.

- _Task<IEnumerable<TResult>> ToListOfAsync<TResult>(
    Expression<Func<T, TResult>> selector,
    Expression<Func<T, bool>> predicate);_
  - _selector_ projects each element of a sequence into a new form.
  - _predicate_ is filter and used along with _Where_.
  - operates asynchronously.
