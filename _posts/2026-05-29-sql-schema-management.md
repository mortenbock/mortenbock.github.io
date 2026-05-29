---
layout: post
title: Managing an evolving sql schema. A declarative approach.
excerpt: A look into using a declarative approach to deploying sql schema changes.
---

For many years, I have been used to applications evolving therir sql schema via migrations. Different flavors like FluentMigrator or EF Core migrations, but essentially the same concept. A stream of instructions that modify a database.

It's a fairly easy concept to understand, and it does have a nice design time feeling letting you feel in full control of what is actually going to happen to the schema when these changes are deployed.

It can have some challenges though, especially as a code base ages.

- Long lists of migrations
- Artifacts describing tables that no longer exist
- Migrations being misused as "run something once" completely unrelated to sql schema
- Drift between the actual database, and the result of running migrations from scratch (someone made a direct change in SQL to fix a prod issue)

I'm sure there are more, and yes, as always, these come down to not having the perfect process. But we're all human (for now).

Still, it made me wonder if there was a different way to approach this. Then I saw a talk by [Erik Ejlskov Jensen](https://github.com/erikej) introducing me to a different set of tools for MsSQL/SQL Server.

## Enter the Sql Project

Using Microsofts own Sql Project, or the OSS [MSBuild.Sdk.SqlProj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj) takes a different approach.

You define, in SQL, how you would like the database schema to look, and then you use the tooling when deploying to compare the desired state, to the current state of the target DB, and it will tell you what needs to change, and apply those changes.

A few reasons why I like this approach:

### More coherent file structure

For each table (and other objects), you will have a single `MyTable.sql` file, that contains the desired design for that table. This means it's easy to see the full schema for it, and it's easy to track historic changes to it via git.

I also prefer this over EF Code First migrations, where it is less obvoius how you need to design your model, to make the table exactly how you want it.

This is just native SQL.

```sql
-- MyTable.sql
CREATE TABLE MyTable
(
    Foo CHAR (10) NOT NULL,
    Bar VARCHAR (50) NOT NULL
);
```

### Drift detection

Using the tooling for SQL server in your deployment process, you can easily get warnings if the target database contains objects that your desired state does not. And if applying those changes would destroy data.

The tools can both generate a report of what changes are detected, and a SQL script to execute them that you can then review before applying.

### Analyzers for good practices

Using the [MSBuild.Sdk.SqlProj](https://github.com/rr-wfm/MSBuild.Sdk.SqlProj) style project, you can take advantage of a large set of analyzers that will let you know if you are making some typical mistakes that could hurt your database design.

It can also tell you if you are not following the naming schemes, and lots of nice reminders like that.

This is typically something that migrations do not give you, because they might only compile to SQL at runtime.

## Should you use it?

If you are seeing some of the issues I mentioned with traditional migration processes, this might be worth a look.

I don't know if MySQL and PostgreSQL have similar tooling options, but I imaging that companies like RedGate offer tools that acheive the same effect.

I found it much cleaner to just declare the desired state, and rely on good tools to apply them, over managing a changelog of diffs.