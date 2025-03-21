First, you will create a rule that assigns users to either an `admin` role, or a single `user` role. Go to the <a href="${manage_url}/#/rules/new" target="_blank" rel="noreferrer">New Rule</a> page on the Auth0 dashboard and select the **Set Roles To A User** template, under **Access Control**.

By default, this rule will assign the user an `admin` role if their email contains `@example.com`. Otherwise, they will be assigned a regular `user` role.

You can modify this line in the default script to change the domain name to one suitable for your setup:

`if (user.email.indexOf('@example.com') > -1)`

::: note
You can set roles other than `admin` and `user` or customize the rule as needed.
:::
