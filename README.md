A Python script to convert Hasura APIs from 🐍snake_case to 🐪camelCase using aliases through Hasura's metadata API + introspection.

## Quick Start
You can run the script as a copy using Google Colab here:
https://colab.research.google.com/drive/1LYajKSAqg6FEruDfWSVN4RB1JfpWdH9A?usp=sharing

Please read and test with your setup first before running on your project.

## How does it work?
1. Finds all fields with `_aggregate` at the end
2. De-camels any fields which already are camelCased (ie. if there's a field like `alreadyCamelCase_Aggregate`)
    - This step is optional
3. Explode the words in snake_case
4. Convert the last word of the phrase to plural + singular using `inflect`
5. Re-join the words in camelCase
6. Build the payload
    - If you look for the `custom_root_fields` JSON build steps you can modify those to customize your naming convention
7. Send to the `/v1/metadata` API

## Example Output
```
Example for => Table Name : active_user
---------------------------------
Before                   : After
active_user              : activeUsers
active_user_by_pk        : activeUser
active_user_aggregagte   : activeUsersAggregate
active_user_insert       : activeUsersInsert
active_user_insert_one   : activeUserInsert
active_user_update       : activeUsersUpdate
active_user_update_by_pk : activeUserUpdate
active_user_delete       : activeUsersDelete
active_user_delete_by_pk : activeUserDelete
```

## API Versions
This version is built for the 2.0+ `pg_set_table_customization` API which supports multiple data sources (which this script isn't currently handling - it would probably need something like the below message where we'd get table names via runSQL) [https://hasura.io/docs/latest/graphql/core/api-reference/metadata-api/table-view.html#pg-set-table-customization]

To back port it to 1.3.4, pretty much all the logic would be the same minus some minor tweaks to the JSON payload which is being sent using `set_table_customization` instead [https://hasura.io/docs/latest/graphql/core/api-reference/schema-metadata-api/table-view.html#set-table-customization].

## Feedback?
This was created pretty quickly so feel free to let me know if there are any improvements you see.

The one edge case that I can imagine is if someone has already aliased their root field with a new name which also has `_aggregate` in it. This would fail for that field since the `/v1/metdata` API is expecting a table name with the reference (and the alias would be different from the table name).

A fix for this would be to use the runSQL API (https://hasura.io/docs/latest/graphql/core/api-reference/schema-metadata-api/run-sql.html) to check that all fields matched database tables / views beforehand -- it seemed like a very infrequent edge-case at the time, I may update that as a later update.
