---
title: The Repository and Unit of Work pattern
date: 2016-10-24T22:30:54+03:00
tags:
  - repository
  - unitofwork
  - pattern
series: Pattern series
categories:
  - repository
  - unitofwork
  - pattern
---

Yes, i know not this again. Is this not the one millionth time that someone blogs about that?
Yes, yes and yes but…
It is always good to repeat things and we all know that

“Repetition is the mother of learning, the father of action, which makes it the architect of accomplishment.” ― [Zig Ziglar](https://en.wikipedia.org/wiki/Zig_Ziglar)

There are still implementations out there that might benefit from this…

So let’s start with some definitions.

## Repository

Quoting [Martin Fowler’s Definition](http://martinfowler.com/eaaCatalog/repository.html):

A system with a complex domain model often benefits from a layer, such as the one provided by Data Mapper (165),
that isolates domain objects from details of the database access code. In such systems it can be worthwhile to build another layer of abstraction
over the mapping layer where query construction code is concentrated. This becomes more important when there are a large number of domain classes or heavy querying.
In these cases particularly, adding this layer helps minimize duplicate query logic. A Repository mediates between the domain and data mapping layers,
acting like an in-memory domain object collection. Client objects construct query specifications declaratively and submit them to Repository for satisfaction.
Objects can be added to and removed from the Repository, as they can from a simple collection of objects, and the mapping code encapsulated by the Repository
will carry out the appropriate operations behind the scenes. Conceptually, a Repository encapsulates the set of objects persisted in a data store and the operations
performed over them, providing a more object-oriented view of the persistence layer. Repository also supports the objective of achieving a clean separation and
one-way dependency between the domain and data mapping layers.

Reading different sources ([MSDN The Repository Pattern](https://msdn.microsoft.com/en-us/library/ff649690.aspx),
[Martin Fowler: Repository](http://martinfowler.com/eaaCatalog/repository.html) etc) about the repository pattern the following properties emerge:

* It maps between Domain Objects and Data objects
* It does not expose the data layer to the outside world
* It consolidates all data access patterns in one place thus help with code deduplication
* It has a single responsibility
* It is simple to implement
* It has a one way dependency between the domain and the data layer

A simple example is the following application repository(C#):

```csharp
public interface IApplicationRepository
{
    Task DeleteAsync(int id);
    Task<ApplicationModel> GetAsync(int id);
}
```

By providing an interface we can leave the implementation up top the developer to choose the data access library they wish.
The argument and return values of this interface should be domain specific objects and not the data objects to avoid spilling the data
into other layers and have a clean separation.

By using the above i had the chance to change the underlying implementation with anything i wished to experimented with.
First everything was EF, then Simple.Data then Dapper etc. You could even mix and match any of the above since every implementation
in the end will use a SqlConnection. It is really easy to change the underlying implementation.

You may think that changing the implementation happens not that often (migrate from EF to Dapper or from nHibernate to EF or Dapper etc)
but it can happen and is a really cheap abstraction over your data layer implementation. It further promotes clean separation which is always something worth doing.
This allows the application to not depend directly on the data access library and allows for future change with little cost.
For example if you have a application that uses nHibernate, which was maybe a good choice in the past, you are missing out some things
that other ORM provide like async calls or even the new .Net Core which may or may not happen for nHibernate. Dapper and EF already have the above.

The implementation of the interface does need something in order to work with the data layer. This can be a SqlConnection, DbContext (EF), Session (NHibernate) etc.
This will be injected to each repository and will generally be implemented in the Unit of Work.

## Unit of Work

Quoting [Martin Fowler's Definition](http://martinfowler.com/eaaCatalog/unitOfWork.html):

A Unit of Work keeps track of everything you do during a business transaction that can affect the database.
When you're done, it figures out everything that needs to be done to alter the database as a result of your work.

So the UoW (Unit of Work) is responsible for keeping the db object (SqlConnection, DbContext) and handling the final commit in order to persist everything to DB.

A simple interface (C#) that has to be implemented is the following:

```csharp
public interface IUnitOfWork : IDisposable
{
    IApplicationRepository Applications { get; }
    Task CommitAsync();
}
```

This is just a wrapper around our db object (SqlConnection, DbContext etc) and the implementation of the commit.
When we have a UoW we have at our hands all the necessary repositories, so interacting with them is really easy.

## EF baked implementation and usage

Now we have the the following implementation for the application repository

```csharp
public class ApplicationRepository : IDataAccess<ApplicationDbModel>, 
                                        IApplicationRepository
{
    public ApplicationRepository(DbContext dbContext, IMapper mapper) : 
        base(dbContext, mapper)
    {
    }

    public Task DeleteAsync(int id) => base.DeleteAsync(id);

    public async Task<ApplicationModel> GetAsync(int id)
    {
            var application = await GetAll()
                        .Where(p => p.Id == id)
                        .SingleOrDefaultAsync();
            return mapper<ApplicationModel>(application);
    }
}
```

Where the base repository is a EF implementation of the following interface:

```csharp
public interface IDataAccess<T> where T : class
{
    IQueryable<T> GetAll();
    Task<T> GetByIdAsync(params object[] keyValues);
    void Add(T entity);
    void Update(T entity);
    void Delete(T entity);
    Task DeleteAsync(params object[] keyValues);
}
```

It is fairly easy to implementing another data access library. A dapper implementation of the application repository has
as constructor parameter a SqlConnection and the actual implementation of the interface methods. That’s it.

The unit of work implementation is the following:

```csharp
public sealed class UnitOfWork : IUnitOfWork
{
    private readonly IMapper _mapper;
    private DbContext _dbContext;

    public UnitOfWork(DbContext dbContext, IMapper mapper)
    {
        _dbContext= dbContext;
        _mapper = mapper;
    }
        
    public Task<int> CommitAsync()
    {
        return _dbContext.SaveChangesAsync();
    }

    public IApplicationRepository Applications => 
        new ApplicationRepository(_dbContext, _mapper);
        
    //Implement IDisposable
}
```

This is a simple implementation of the UoW. Do not mind that some features are missing like transaction handling
(DbContext.Database.BeginTransaction() and then commit or rollback) a repository factory etc which are fairly easy to implement.

And how is this used?

Let’s assume we have a Unit Of Work Factory implemented so the code would be:

```csharp
using (var uow = _unitOfWorkFactory.Create())
{
    var application = await uow.Applications.GetAsync(1, 1);
    await uow.Applications.DeleteAsync(application.Id);
    await uow.CommitAsync();
}
```

Easy and clean, isn’t it? Everything is in one place, at the end it get’s committed and properly disposed.
Since EF exposes the connection through the DbContext we can actually use Dapper also and have a mixed data access layer repository
in order to handle some hotspots where EF does not play well.

## Conclusion

The repository and the unit of work patterns are fairly easy to implement. They provide a proper data access abstraction and expose only the needed domain object
and do not spill the data object into the upper layers. The only thing needed in order to use this is to inject the unit of work factory and we have our db in our hand.
Hope this is helpful. Any comment, discussion or fix is highly welcome.
