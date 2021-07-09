---
title: "Recursively deleting all protected related objects in Django"
date: 2021-07-09T10:48:12-03:00
draft: false
---

When debugging Django applications, it's often useful to delete objects so that
you can try the thing you're working on again and recreate them. This isn't
always trivial though, because related objects that have `on_delete=PROTECT` can
prevent you from doing so.

This is a great safeguard when the application is running in production, but not
so much when you know what you're doing and just want to test something in your
own environment.

I was surprised to find that there were no StackOverflow answers or gists for a
quick way to just let me do that, so
I [provided my own](https://stackoverflow.com/a/68317854/5686598). Here it is:

{{< gist pedrovhb 18dc8b7df90d2310c354af98c3e33e23 >}}

I need to reinforce this - **this isn't a good thing to use in production**.
Relationships aren't always obvious and if something is protected, it's probably
for a reason. I'd only use this to help with debugging in one-off scripts (or on
the shell) with test databases that I don't mind wiping.

<details>
  <summary>
        <i>in case the gist is down, click this text to see the snippet</i>
  </summary>

```python
def recursive_delete(to_del):
    """Recursively delete an object, all of its protected related
    instances, those instances' protected instances, and so on.
    """
    from django.db.models import ProtectedError
    while True:
        try:
            to_del_pk = to_del.pk
            if to_del_pk is None:
                return  # unsaved object
            to_del.delete()
            print(f"Deleted {to_del.__class__.__name__} with pk {to_del_pk}: {to_del}")
        except ProtectedError as e:
            for protected_ob in e.protected_objects:
                recursive_delete(protected_ob)
```
</details>

