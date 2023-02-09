---
title: Immutable classes for API data (a practical advice)
date: 2022-02-09 17:26:00
tags: [advice]
categories: [Articles]
---

Let's say you're programming an online advertisement system for a fastfood restaurants corporation 🍟🍔🍕🥤.
You need to show the top-5 most suitable meals. There is a pretty hard formula of suitability that
takes into account the user's region, meals availability in the closest restaurant, current promo
campaigns, the current time, and so on.

There are HTTP requests that come into the API of your service. Let's imagine we have a C++ class/struct for every
API entity.

Look at the `Restaurant` API class that describes one of the closest restaurant. It holds the distance to the user
(the further away it from the user, the worser its "weight" in the formula), the occupancy (workload),
the list of available meals, the current promo campaigns, and the visit history for the current user.
```c++
struct Restaurant {
    double Distance;
    double Occupancy;
    std::vector<Meal*> Meals;
    std::vector<Promo*> Promos;
    std::vector<Visit> VisitHistory;
};
```
The object *owns* some data (like visits history), and *references* some other data (because it's a common data, like
the description of meals).

There comes a problem - let's say we have a `std::vector<Meal> Meals` somewhere. If we created some `Restaurant`s,
and then added a new `Meal` to this container, we can provoke a vector reallocation, and all `Meal*` pointers become
dandling.

It's possible to wrap the object with a smart pointer `std::vector<std::shared_ptr<Meal>> Meals`,
but it's not free and looks awful.

There is an approach that solves these problems - **every API class should be declared as [non-copyable](https://www.boost.org/doc/libs/master/libs/core/doc/html/core/noncopyable.html#core.noncopyable.header_boost_core_noncopyable_hp) and non-movable**. Which advantages it brings:
1. It's impossible to accidentally pass these objects by value or to copy them.
2. Pointers to these objects live as long as the container which holds these objects lives.

The C++ compiler won't allow to use containers like `std::vector` that could potentially invalidate pointers.
The code will compile only when using "safe" containers like `std::list`.
