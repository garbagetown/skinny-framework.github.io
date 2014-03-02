---
title: Scaffolding - Skinny Framework
---

## Scaffolding

<hr/>
### Scaffold Generator
<hr/>

Skinny has a powerful scaffold generator. Usage is very simple.

```
./skinny g scaffold members member name:String "nickname:String:varchar(64)" birthday:Option[LocalDate]
./skinny db:migrate
./skinny run
```

Now you can use CRUD pages for members resource at `http://localhost:8080/members`.

<hr/>
#### Parameters

Scaffold command's parameters are ...

```
./skinny g scaffold:{template} {resources} {resource} "{fieldName:paramType(:columnType)}" ...
```

- {template}: "ssp" / "scaml" / "jade" (Scalate template name)
- {resources}: Resource name in the plural (camelCase)
- {resource}: Resource name (camelCase)
- {fieldName}: Field name in the resource (camelCase)
- {paramType}: `skinny.ParamType`. see also: [skinny/ParamType.scala](https://github.com/skinny-framework/skinny-framework/blob/develop/common/src/main/scala/skinny/ParamType.scala)
- {columnType}: (optional) Database column type. This will be embedded into DB migration file.

<hr/>
#### Example Usage
<hr/>

Let's create a CRUD pages for project members!

<hr/>

```
$ ./skinny g scaffold:jade projectMembers projectMember name:String "nickname:Option[String]:varchar(32)" joinedAt:DateTime leaveAt:Option[DateTime]

[info] Loading global plugins from /Users/seratch/.sbt/0.13/plugins
[info] Updating {file:/Users/seratch/.sbt/0.13/plugins/}global-plugins...
[info] Resolving org.fusesource.jansi#jansi;1.4 ...
[info] Done updating.
[info] Loading project definition from /home/project
[info] Set current project to dev (in build file:/home/)
[info] Generating /home/task/target/scala-2.10/resource_managed/main/rebel.xml.
[info] Running TaskRunner generate:scaffold:jade projectMembers projectMember name:String nickname:Option[String]:varchar(32) joinedAt:DateTime leaveAt:Option[DateTime]

 *** Skinny Generator Task ***

  "src/main/scala/controller/ApplicationController.scala" skipped.
  "src/main/scala/controller/ProjectMembersController.scala" created.
  "src/main/scala/ScalatraBootstrap.scala" modified.
  "src/test/scala/controller/ProjectMembersControllerSpec.scala" created.
  "src/test/resources/factories.conf" modified.
  "src/main/scala/model/ProjectMember.scala" created.
  "src/test/scala/model/ProjectMemberSpec.scala" created.
  "src/main/webapp/WEB-INF/views/projectMembers/_form.html.jade" created.
  "src/main/webapp/WEB-INF/views/projectMembers/new.html.jade" created.
  "src/main/webapp/WEB-INF/views/projectMembers/edit.html.jade" created.
  "src/main/webapp/WEB-INF/views/projectMembers/index.html.jade" created.
  "src/main/webapp/WEB-INF/views/projectMembers/show.html.jade" created.
  "src/main/resources/messages.conf" modified.
  "src/main/resources/db/migration/V20140302005853__Create_projectMembers_table.sql" created.

[success] Total time: 1 s, completed Mar 2, 2014 12:58:53 AM
```

<hr/>
After that, do DB migration.
<hr/>

```
$ ./skinny db:migrate

[info] Loading global plugins from /Users/seratch/.sbt/0.13/plugins
[info] Loading project definition from /home/project
[info] Set current project to dev (in build file:/home/)
[info] Generating /home/task/target/scala-2.10/resource_managed/main/rebel.xml.
[info] Running TaskRunner db:migrate
2014-03-02 01:01:00,305 DEBUG [run-main-0] s.ConnectionPool$ [Log.scala:45] Registered connection pool : ConnectionPool(url:jdbc:h2:file:db/development, user:sa)
Mar 02, 2014 1:01:00 AM com.googlecode.flyway.core.command.DbMigrate migrate
INFO: Current version of schema "PUBLIC": 20140301231655
Mar 02, 2014 1:01:00 AM com.googlecode.flyway.core.command.DbMigrate applyMigration
INFO: Migrating schema "PUBLIC" to version 20140301231759
Mar 02, 2014 1:01:00 AM com.googlecode.flyway.core.command.DbMigrate applyMigration
INFO: Migrating schema "PUBLIC" to version 20140302005853
Mar 02, 2014 1:01:00 AM com.googlecode.flyway.core.command.DbMigrate logSummary
INFO: Successfully applied 2 migrations to schema "PUBLIC" (execution time 00:00.078s).
2014-03-02 01:01:01,043 DEBUG [run-main-0] s.ConnectionPool$ [Log.scala:45] Registered connection pool : ConnectionPool(url:jdbc:h2:file:db/development, user:sa)
2014-03-02 01:01:01,045 DEBUG [Thread-3] s.ConnectionPool$ [Log.scala:45] The old pool destruction started. connection pool : ConnectionPool(url:jdbc:h2:file:db/development, user:sa)
2014-03-02 01:01:01,067 DEBUG [Thread-3] s.ConnectionPool$ [Log.scala:45] The old pool is successfully closed. connection pool : ConnectionPool(url:jdbc:h2:file:db/development, user:sa)
[success] Total time: 2 s, completed Mar 2, 2014 1:01:01 AM
```

<hr/>
Run the Skinny app and access `http://localhost:8080/project_members` from your browser.
<hr/>

```
$ ./skinny run
```

<hr/>
#### URL / Parameter Names
<hr/>

The following is an example controller generated by scaffold command.

Be aware of these rules:

- Base URL will be snake_cased. If you'd like to change it, edit `resourcesBasePath`
- Parameter names will be snake_cased. `useSnakeCasedParamKeys` should be true when using `Params#withDateTime`

It'll be the fastest way to understand actual files generated by scaffold generator.

<hr/>

```java
package controller

import skinny._
import skinny.validator._
import model.ProjectMember

object ProjectMembersController extends SkinnyResource with ApplicationController {
  protectFromForgery()

  override def model = ProjectMember
  override def resourcesName = "projectMembers"
  override def resourceName = "projectMember"

  override def resourcesBasePath = s"/${toSnakeCase(resourcesName)}"
  override def useSnakeCasedParamKeys = true

  override def createForm = validation(createParams,
    paramKey("name") is required & maxLength(512),
    paramKey("nickname") is maxLength(512),
    paramKey("joined_at") is required & dateTimeFormat,
    paramKey("leave_at") is dateTimeFormat
  )
  override def createParams = Params(params).withDateTime("joined_at").withDateTime("leave_at")
  override def createFormStrongParameters = Seq(
    "name" -> ParamType.String,
    "nickname" -> ParamType.String,
    "joined_at" -> ParamType.DateTime,
    "leave_at" -> ParamType.DateTime
  )

...
```

<hr/>
### Reverse Scaffold Generator
<hr/>

If you're working with existing database, `reverse-scaffold` command is pretty useful.

When you have `project_members` table in database,

```sql
-- For H2 Database
create table project_members (
  id bigserial not null primary key,
  name varchar(512) not null,
  nickname varchar(32),
  joined_at timestamp not null,
  leave_at timestamp
);
```

Run the `reverse-scaffold` command like this.

This command uses DB settings in the `src/main/resources/application.conf`.

Default SkinnyEnv is `development`. If you need specifying others, add it as the fourth parameter.

```
$ ./skinny g reverse-scaffold:scaml project_members projectMembers projectMember

[info] Loading global plugins from /Users/seratch/.sbt/0.13/plugins
[info] Loading project definition from /home/project
[info] Set current project to dev (in build file:/home/)
[info] Generating /home/task/target/scala-2.10/resource_managed/main/rebel.xml.
[info] Running TaskRunner generate:reverse-scaffold:scaml project_members projectMembers projectMember

 *** Skinny Reverse Engineering Task ***

  Table: project_members
  Resources: projectMembers
  Resource: projectMember

  Columns:
   - name:String:varchar(512)
   - nickname:Option[String]:varchar(32)
   - joinedAt:DateTime
   - leaveAt:Option[DateTime]

 *** Skinny Generator Task ***

  "src/main/scala/controller/ApplicationController.scala" skipped.
  "src/main/scala/controller/ProjectMembersController.scala" created.
  "src/main/scala/ScalatraBootstrap.scala" modified.
  "src/test/scala/controller/ProjectMembersControllerSpec.scala" created.
  "src/test/resources/factories.conf" modified.
  "src/main/scala/model/ProjectMember.scala" created.
  "src/test/scala/model/ProjectMemberSpec.scala" created.
  "src/main/webapp/WEB-INF/views/projectMembers/_form.html.scaml" created.
  "src/main/webapp/WEB-INF/views/projectMembers/new.html.scaml" created.
  "src/main/webapp/WEB-INF/views/projectMembers/edit.html.scaml" created.
  "src/main/webapp/WEB-INF/views/projectMembers/index.html.scaml" created.
  "src/main/webapp/WEB-INF/views/projectMembers/show.html.scaml" created.
  "src/main/resources/messages.conf" modified.
  "src/main/resources/db/migration/V20140302010916__Create_projectMembers_table.sql" created.

[success] Total time: 1 s, completed Mar 2, 2014 1:09:16 AM
```

The DB migration file generated by this command is commented out by default, so `./skinny db:migrate` will do nothing.

Just run `./skinny run` and access `http://localhost:8080/project_members` from your browser.

<hr/>
#### Parameters

Scaffold command's parameters are ...

```
./skinny g reverse-scaffold:{template} {tableName} {resources} {resource} {skinnyEnv}"
```

- {template}: "ssp" / "scaml" / "jade" (Scalate template name)
- {tableName}: Target table name in the existing database
- {resources}: Resource name in the plural (camelCase)
- {resource}: Resource name (camelCase)
- {skinnyEnv}: (optional) default value is "development"
