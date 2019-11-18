# Part 4 - Migrations

## Getting Started

Now that we have our authentication setup and we are comfortable with migrating our migrations, let's create a new migration where we will store our posts.

Our posts table should have a few obvious columns that we will simplify for this tutorial part. Let's walk through how we create migrations with Masonite

## Craft Command

Not surprisingly, we have a craft command to create migrations. You can read more about [Database Migrations here](../orator-orm/database-migrations.md) but we'll simplify it down to the command and explain a little bit of what's going on:

{% code title="terminal" %}
```text
$ craft migration create_posts_table --create posts
```
{% endcode %}

This command simply creates the basis of a migration that will create the posts table. By convention, table names should be plural \(and model names should be singular but more on this later\).

This will create a migration in the `databases/migrations` folder. Let's open that up and starting on line 6 we should see look like:

{% code title="databases/migrations/2018\_01\_09\_043202\_create\_posts\_table.py" %}
```python
def up(self):
    """
    Run the migrations.
    """
    with self.schema.create('posts') as table:
        table.increments('id')
        table.timestamps()
```
{% endcode %}

Lets add a title, an author, and a body to our posts tables.

{% code title="databases/migrations/2018\_01\_09\_043202\_create\_posts\_table.py" %}
```python
def up(self):
    """
    Run the migrations.
    """
    with self.schema.create('posts') as table:
        table.increments('id')
        table.string('title')

        table.integer('author_id').unsigned()
        table.foreign('author_id').references('id').on('users')

        table.string('body')
        table.timestamps()
```
{% endcode %}

{% hint style="success" %}
This should be fairly straight forward but if you want to learn more, be sure to read the [Database Migrations](../orator-orm/database-migrations.md) documentation.
{% endhint %}

Now we can migrate this migration to create the posts table

{% code title="terminal" %}
```text
$ craft migrate
```
{% endcode %}

