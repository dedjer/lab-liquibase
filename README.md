# lab-liquibase
Manual liquibase project

## Install Liquibase

- Windows - choco install liquibase
- Mac - brew install liquibase

## Setup Liquibase - Steps to build this repo from scratch

Create liquibase.properties file using sample code below, or use the one supplied in this repo:

```json
changeLogFile:  dbchangelog.xml  
url:  jdbc:postgresql://localhost:5432/mydatabase
username:  postgres  
password:  password 
classpath:  postgresql-42.2.8.jar
liquibaseProLicenseKey:  licensekey
liquibase.hub.ApiKey:  APIkey
```

Generate changelog file:

NOTE: You can omit the `--schema` option if you prefer to use the default schema or if you've defined the schema in your `liquibase.properties` file

```bash
liquibase --changeLogFile=dbchangelog.xml --schemas=db_example generateChangeLog
```

Add a new changeset to the changelog file:

NOTE: A `schemaName=` parameter is provided to ensure that the table is not created in the default `public` postgres schema

```xml
    <changeSet author="djgarcia" id="4">
        <createTable schemaName="db_example" tableName="liquibase_gen">
            <column autoIncrement="true" name="id" type="INTEGER">
                <constraints nullable="false" primaryKey="true" primaryKeyName="user_pkey"/>
            </column>
            <column name="first_name" type="VARCHAR"/>
            <column name="last_name" type="VARCHAR"/>
            <column name="user_id" type="INTEGER"/>
        </createTable>
    </changeSet>  
```

Run the liquibase update:

```bash
liquibase update
```

## Troubleshooting

If you receive the below error, make sure your postgres user has `CREATE TABLE` permissions

```sql
Unexpected error running Liquibase: ERROR: permission denied for schema db_example
  Position: 14 [Failed SQL: (0) CREATE TABLE db_example.databasechangeloglock (ID INTEGER NOT NULL, LOCKED BOOLEAN NOT NULL, LOCKGRANTED TIMESTAMP WITHOUT TIME ZONE, LOCKEDBY VARCHAR(255), CONSTRAINT databasechangeloglock_pkey PRIMARY KEY (ID))]
```

Migration error

```sql
Unexpected error running Liquibase: Migration failed for changeset dbchangelog.xml::1663521877828-1::djgarcia (generated):
     Reason: liquibase.exception.DatabaseException: ERROR: relation "teams" already exists [Failed SQL: (0) CREATE TABLE db_example.teams (id INTEGER GENERATED BY DEFAULT AS IDENTITY NOT NULL, team_name VARCHAR, CONSTRAINT teams_pkey PRIMARY KEY (id))]
```

The above error occurs if you generate the dbchangelog.xml file from the database and run a `liquibase update`.  Since the tables already exist the udpate will fail. 

Add `preConditions` to each changeset to prevent the migration error, see examples below and in the dbchangelog.xml provided in repo:

NOTE: See documentation here: https://docs.liquibase.com/concepts/changelogs/preconditions.html

```xml
<preConditions onFail="MARK_RAN">
    <not>
        <tableExists schemaName="db_example"  tableName="teams"/>
    </not> 
</preConditions>

<preConditions onFail="MARK_RAN">
    <not>
        <foreignKeyConstraintExists schemaName="db_example" foreignKeyName="players_team_id_fkey"/>
    </not> 
</preConditions>
```