# How to authorize GraphQL operations

GraphQL operations (queries, mutations and subscriptions) need to check various parameters to ensure user cannot do something that is not allowed! Most common scenarios are covered by built-in helpers.

## Allow public access

By default, all operations are accessible to any logged in users. But sometimes guest sessions need access too.

```ruby
class Query
  allow_public_access!
end
```

## Require user permissions

This is a simple current user permissions check. It can take multiple permissions for OR check as well as plus-style for AND check.

```ruby
class Query
  requires_permission 'ticket.agent', 'ticket.customer+something'
end
```

## Require a Setting

Some stuff requires a specific `Setting`. Or, sometimes, specific `Setting` being NOT turned on. Those two helpers can handle that. Both can take a custom error message too! 

```ruby
class Mutation
  requires_enabled_setting 'checklist', error_message: 'Custom Error'
  requires_disabled_setting 'blocker'
end
```

## Use the Pundit, developer!

In many cases, a clever usage of Pundit policies may be the cleanest approach!

For example, we want to check if the user is allowed to add a new item to the checklist. At first sight, we may want to check manually using `ChecklistItemPolicy#create?`. But on the other hand we can simply check if it's OK to update the checklist while loading that object. We'd be checking if we can `show?` the Checklist anyway.

```ruby
class AddChecklistItem < Mutation
  argument :checklist_id, loads: Gql::Types::ChecklistType, loads_pundit_method: :update?

  def resolve(checklist:)
    add_item
  end
end
```

## Neither of above matches my use case!1!!

Sometimes an interesting case pops up and neither of above helps. In such case, please override `def authorized?` **instance** method. Return `true` on success. In case of a failure, there're two legit approaches. Simply return `false`. Or raise `Exceptions::Forbidden` with a custom error message.

Please **do not override** `self.authorized?` class method! Nor the old `self.authorize`!

No need to use `super`! Out-of-box implementation simply returns `true`.

```ruby
class Query
  argument :some_arg

  def authorized?(some_arg:)
    super_duper_custom_logic
  end
end
```

## Disable CSRF check

This is not exactly authorization, but it's still somewhat related. Usually CSRF is required to prevent cross-site forgery attacks. But sometimes there're legit reasons to allow any POST request.

Applies to mutations only!

```ruby
class Mutation
  skip_csrf_verification!
end
```