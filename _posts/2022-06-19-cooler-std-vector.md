---
title: std::vector, but 20% cooler
date: 2022-06-19 16:32:00
tags: [advice]
categories: [Article]
---

It is often required to work on a group of objects, to somehow sort them, to find the object by a key, and so on - depending
on the business logic.

Let's say we have objects of `Meal` type🍝, then you'd represent a number of meals like this:
```c++
std::vector<Meal> meals;
```

And you'd have functions to do what is needed:
```c++
void SortAlphabetically(std::vector<Meal>& meals);
void SortByCostDesc(std::vector<Meal>& meals);
const Meal* FindMealById(const std::vector<Meal>& meals, std::string_view id);
```

This approach reduces the code readability, because a group of objects kind of lives independently of their methods.

I have applied an approach - to **inherit from std::vector** and define the needed methods there. It looks like that:
```c++
class MealList : public std::vector<Meal> {
public:
    static MealList ParseFromJson(const nlohmann::json& json);

    void SortAlphabetically();
    void SortByCostDesc();
    const Meal* FindMealById(std::string_view id) const;

private:
    MealList() = default;  

};
```
And the methods' definitions are in the corresponding `.cpp`-file.

The `MealList` holds all the needed methods and preserves the `std::vector` semantics
(for example, you can do use a [range-based for loop](https://en.cppreference.com/w/cpp/language/range-for) on an object of this type).

P. S. They say that inheriting from a class of the `std` namespace might be an UB, but I don't quite belive it, call me a risky guy.
