---
title: Data-Oriented Inheritance Challenges
date: 2025-12-15
tags: [c#, inheritance]
---

## The Problem

**"I have 15 classes and they all do the same thing... why did I do this?"**

Disclaimer: *This post does not advocate for or against inheritance or composition in general. I'd like to focus specifically on why using inheritance solely to represent different data values can be suboptimal, and why some consider an anti-pattern. I'll also talk about alternative approaches. This was inspired by lessons learned while learning C# with the [C# Player's Guide - RB Whitaker](https://csharpplayersguide.com/) for classes, enums, inheritance, and extracting strings from enums.*

Audience: Beginner to early-intermediate/novice.

In C# you may find yourself needing to create objects that share similarities. A simple example of this in games is monsters and items. For this exercise, let's stick with monsters.

We start with a simple scenario: Our game has 3 different types of monsters: **Goblins**, **Skeletons**, and **Ogres**. Each needs a name and starting hit points (HP):

- **Goblin**: 5 HP
- **Skeleton**: 7 HP 
- **Ogre**: 15 HP

How should we model this in C#?

In this article we’ll see why creating a whole class for each monster type is undesirable when the _only difference between monsters is data_. We'll then look at simpler and safer ways to store it.

## Working with Enums
Taking our three monster types, we can quickly represent them using an enumeration:
```cs
Monster goblin = Monster.Goblin;
Console.WriteLine(goblin);

public enum Monster { Goblin, Skeleton, Ogre }
```

**Console Output:**
```console
Goblin
```

As a basic start we are simply using an enumeration that can easily represent the monster names.

By default, passing an enum value to `Console.WriteLine()` prints the enum member name as a string because `WriteLine()` calls `ToString()` internally.

But what happens if our enum names contain multiple words? Let's add a few more monsters to illustrate that:
```cs
Monster ogre = Monster.Ogre;
Console.WriteLine(ogre);

Monster venomousSpider = Monster.VenomousSpider;
Console.WriteLine(venomousSpider);

public enum Monster { Goblin, Skeleton, Ogre, FireElemental, VenomousSpider }
```

**Console Output:**
```console
Ogre
VenomousSpider
```

Now we can see a limitation: printing the enum value gives us the member name verbatim, which may not be formatted as a *user-friendly* string. This can be problematic when displaying names to users or sending them to external systems (like files, databases, or even a UI).

Let's explore a solution for getting properly formatted display names.

### Getting Strings from Enums
Using the following enum:
```cs
public enum Monster { Goblin, Skeleton, Ogre, FireElemental, VenomousSpider }
```

We can use a switch expression to map enum values to formatted display strings. The method accepts an enum value as a parameter and returns the formatted string:
```cs
Console.WriteLine(GetMonsterDisplayName(Monster.Ogre));
Console.WriteLine(GetMonsterDisplayName(Monster.FireElemental));
Console.WriteLine(GetMonsterDisplayName(Monster.VenomousSpider));

string GetMonsterDisplayName(Monster monster)
{
    return monster switch
    {
        Monster.Goblin => "Goblin",
        Monster.Skeleton => "Skeleton",
        Monster.Ogre => "Ogre",
        Monster.FireElemental => "Fire Elemental",
        Monster.VenomousSpider => "Venomous Spider",
        _ => "NO NAME SET"
    };
}
```

**Console Output:**
```console
Ogre
Fire Elemental
Venomous Spider
```

Now we have properly formatted display names and reusable code! The trade-off is **maintenance**. Whenever we add a new monster to the enum, we must also add a corresponding case to the switch expression. Failing to do this will result in returning "NO NAME SET" for otherwise valid monsters.

I recommend heading to this article for further reading on parsing enumerations: [RB Whitaker: Parsing Enumerations](https://csharpplayersguide.com/blog/2022/05/22/parsing-enumerations/).

## Classes and Data
The previous section showed one way to get a string from an enum. There are other approaches, but they're outside the scope of this article. More importantly, we haven't addressed how to store starting HP values for each monster.

Consider the below code and you can see why programmers begin moving away from enums when they need more data:
```cs
Monster goblin = Monster.Goblin;
string goblinName = GetMonsterDisplayName(goblin);
int goblinStartingHP = 5;

string GetMonsterDisplayName(Monster monster)
{
    return monster switch
    {
        Monster.Goblin => "Goblin",
        Monster.Skeleton => "Skeleton",
        Monster.Ogre => "Ogre",
        Monster.FireElemental => "Fire Elemental",
        Monster.VenomousSpider => "Venomous Spider",
        _ => "NO NAME SET"
    };
}
public enum Monster { Goblin, Skeleton, Ogre, FireElemental, VenomousSpider }
```

You could argue that you'd want yet another method to return an `int` from a `Monster` enum value, similar to how we did with strings:
```cs
int GetMonsterStartingHP(Monster monster)
{
    return monster switch
    {
        Monster.Goblin => 5,
        Monster.Skeleton => 7,
        Monster.Ogre => 15,
        Monster.FireElemental => 10,
        Monster.VenomousSpider => 12,
        _ => 1
    };
}
```

Now we can call it like this:
```cs
Monster goblin = Monster.Goblin;
string goblinName = GetMonsterDisplayName(goblin);
int goblinHealth = GetMonsterStartingHP(goblin);
```

This approach becomes increasingly time-consuming to maintain. Every new monster requires updating both switch expressions (name and starting HP), and every new property means adding yet another mapping method. It’s tedious, error-prone, and easy to break as the codebase grows.

Next, let’s explore how we can try to solve this by combining enums with classes.

### Using a Class
The solution is to create a class that stores monster data while using an enum for the monster type identification. First, we need to rename our enum to avoid a naming conflict:
```cs
public enum MonsterType { Goblin, Skeleton, Ogre, FireElemental, VenomousSpider }
```

Define our class so we have a `Name`, `StartingHP`, and a `MonsterType`:
```cs
public class Monster
{
    public MonsterType MonsterType { get; }
    public string Name { get; }
    public int StartingHP { get; }

    public Monster(MonsterType monsterType, string name, int startingHP)
    {
        MonsterType = monsterType;
        Name = name;
        StartingHP = startingHP;
    }
}
```

Now we can instantiate Monsters:
```cs
Monster fireElemental = new(MonsterType.FireElemental, "Fire Elemental", 10);
Monster venomousSpider = new(MonsterType.VenomousSpider, "Venomous Spider", 12);
Monster skeleton = new(MonsterType.Skeleton, "Skeleton", 7);

Console.WriteLine($"{fireElemental.Name}: {fireElemental.StartingHP} HP"); 
Console.WriteLine($"{venomousSpider.Name}: {venomousSpider.StartingHP} HP"); 
Console.WriteLine($"{skeleton.Name}: {skeleton.StartingHP} HP");
```

**Console Output:**
```console
Fire Elemental: 10 HP
Venomous Spider: 12 HP
Skeleton: 7 HP
```

This is now a significant improvement over maintaining separate methods that are not encapsulated within a class, and it greatly simplifies it while reducing the fragility of our code base when we want to create monsters. Also, all of the data is stored on the object rather than variables floating around in the code we have to keep track of.

But... what prevents us from:
- Forgetting the name of each type?
- Having a typo with the name?
- Setting a StartingHP value that would potentially be game-balance breaking? What about a negative value?
- Accidentally creating a `MonsterType.FireElemental` but calling it an "Ogre"?

### Temptation of Inheritance
After working with code like this a bit, or running into a random bug, you may wonder: "What if each of the monsters were their own class that contains definitions for all of these different pieces of data?"

## Using Inheritance
With inheritance we can declare a base class, and even make it `abstract` which means that you will be unable to make an instance of that class and instead it's just used to model the derived classes. Additionally, we can mark the properties as abstract, which will enforce any derived classes to supply their own data:
```cs
public abstract class Monster
{
    public abstract string Name { get; }
    public abstract int StartingHP { get; }
}
public class Goblin : Monster
{
    public override string Name => "Goblin";
    public override int StartingHP => 5;
}
public class Skeleton : Monster
{
    public override string Name => "Skeleton";
    public override int StartingHP => 7;
}
public class Ogre : Monster
{
    public override string Name => "Ogre";
    public override int StartingHP => 15;
}
public class FireElemental : Monster
{
    public override string Name => "Fire Elemental";
    public override int StartingHP => 10;
}
public class VenomousSpider : Monster
{
    public override string Name => "Venomous Spider";
    public override int StartingHP => 12;
}
```

Now we can be a bit more direct with our declarations and let the discrete classes handle the data:
```cs
Skeleton skeleton = new();
FireElemental fireElemental = new();

Console.WriteLine($"{fireElemental.Name}: {fireElemental.StartingHP} HP"); 
Console.WriteLine($"{skeleton.Name}: {skeleton.StartingHP} HP");
```

With this design we avoid using fragile constructor calls that may contain name typos, invalid starting HP values, or mismatched MonsterType.

However, a drawback becomes quickly apparent. What if the game added a new biome that warrants adding seven new monster types? We've now got a **class explosion** problem. We must manually create a new class for every new monster type we want in the game. This problem becomes worse if we adopt this same pattern for other types: Weapons, Armor, Items, Attacks, Spells, Attributes, etc. The number of classes grows rapidly.

Even **worse**:
- Adding one new monster type means adding one new class
- Adding one new data property to the base class means updating every existing subclass

This is a lot of boilerplate code where the only purpose is to hold different data and not different behavior. When inheritance is used only to represent different static values, the maintenance cost grows **linearly** for new types and **exponentially** for new properties.

## Solution(s)
```cs
public class Monster
{
    public MonsterType MonsterType { get; }
    
    public string Name => MonsterType switch
    {
        MonsterType.Goblin => "Goblin",
        MonsterType.Skeleton => "Skeleton",
        MonsterType.Ogre => "Ogre",
        MonsterType.FireElemental => "Fire Elemental",
        MonsterType.VenomousSpider => "Venomous Spider",
        _ => "NO NAME SET"
    };
    
    public int StartingHP => MonsterType switch
    {
        MonsterType.Goblin => 5,
        MonsterType.Skeleton => 7,
        MonsterType.Ogre => 15,
        MonsterType.FireElemental => 10,
        MonsterType.VenomousSpider => 12,
        _ => 1
    };

    public Monster(MonsterType monsterType)
    {
        MonsterType = monsterType;
    }
}
```

Yes, we are using switch expressions again. However, unlike before all logic is centralized in one class rather than scattered across multiple classes or methods. Adding a new property now requires updating only one class, not many (dozens or even hundreds) of subclasses.

The benefits of this approach:
- `Name` and `StartingHP` are computed properties: Their values are derived from the `MonsterType`.
- Type-safe creation: You cannot create invalid monster types.
- Single source of truth: All monster related logic lives in one class.

Now we can create new monsters in our program by simply passing in the `MonsterType` and the logic in the class definition handles the rest:
```cs
Monster fireElemental = new(MonsterType.FireElemental);
Monster venomousSpider = new(MonsterType.VenomousSpider);
Monster ogre = new(MonsterType.Ogre);

Console.WriteLine($"{fireElemental.Name}: {fireElemental.StartingHP} HP"); 
Console.WriteLine($"{venomousSpider.Name}: {venomousSpider.StartingHP} HP"); 
Console.WriteLine($"{ogre.Name}: {ogre.StartingHP} HP");
```

**Console Output:**
```cs
Fire Elemental: 10 HP
Venomous Spider: 12 HP
Ogre: 15 HP
```

### When Inheritance is the Correct Choice
Use inheritance when derived classes should have different behaviors from other derived classes of the same base class. Realistically we'd have more properties such as CurrentHP, etc. but we will skip the verbosity.
```cs
public abstract class Monster
{
    public abstract string Name { get; }
    public abstract int StartingHP { get; }
    public abstract void Attack();
    public abstract void TakeDamage();

}
public class VenomousSpider : Monster
{
    public override string Name => "Venomous Spider";
    public override int StartingHP => 12;

    public override void Attack()
    {
        Console.WriteLine($"The {Name} bites with fangs dripping with venom!");
    }
    public override void TakeDamage()
    {
        // damage log when damage is taken. Maybe it's resistant to physical damage?
    }
}

public class FireElemental : Monster
{
    public override string Name => "Fire Elemental";
    public override int StartingHP => 10;

    public override void Attack()
    {
        Console.WriteLine($"The {Name} launches a blazing fireball!");
    }
    public override void TakeDamage()
    {
        // Maybe it's IMMUNE to fire damage?
        // Maybe it takes DOUBLE damage from cold?
    }
}

public enum DamageType { Physical, Cold, Fire }
```

Now we start to see the benefits of inheritance with polymorphism. We no longer need the `MonsterType` enum because the type is inferred from the class names, and we can now define data inside each of the classes but now we have hte benefit of different behaviors. The extra work of declaring additional classes for each monster type is worth the effort.

### Optional Considerations

#### Static Factory Method
A consideration can be made if the program is small to create static factory methods using the same class structure above:
```cs
public class Monster
{
	// previous class properties
	
    public Monster(MonsterType monsterType, string name, int startingHP)
    {
        MonsterType = monsterType;
        Name = name;
        StartingHP = startingHP;
    }
    public static Monster CreateGoblin() => new(MonsterType.Goblin, "Goblin", 5);
    public static Monster CreateSkeleton() => new(MonsterType.Skeleton, "Skeleton", 7);
    public static Monster CreateOgre() => new(MonsterType.Ogre, "Ogre", 15);
    public static Monster CreateFireElemental() => new(MonsterType.FireElemental, "Fire Elemental", 10);
    public static Monster CreateVenomousSpider() => new(MonsterType.VenomousSpider, "Venomous Spider", 12);
}
public enum MonsterType { Goblin, Skeleton, Ogre, FireElemental, VenomousSpider }
```

Now our instantiations are cleaner and less error-prone:
```cs
Monster goblin = Monster.CreateGoblin();
Monster skeleton = Monster.CreateSkeleton();
Monster ogre = Monster.CreateOgre();
Monster fireElemental = Monster.CreateFireElemental();
Monster venomousSpider = Monster.CreateVenomousSpider();
```

This design is perfectly reasonable as a stepping-stone, but it still will have the problem when we want to create a new property. You'll have to then go back into each static method and update it with the new parameter.

#### Using a Record
Another alternative is to define `Monster` as a **record**. These are ideal for data containers because records are **immutable by default** and automatically provide equality checks, `ToString()`, and other features. The intricate details of Records are outside the scope here.
```cs
public record Monster(MonsterType monsterType, string Name, int StartingHP);
```

Example usage:
```cs
Monster fireElemental = new(MonsterType.FireElemental, "Fire Elemental", 10);
Monster ogre = new(MonsterType.Ogre, "Ogre", 15);

Console.WriteLine($"{fireElemental.Name}: {fireElemental.StartingHP} HP");
Console.WriteLine($"{ogre.Name}: {ogre.StartingHP} HP");
```

Consequently, using a record still requires passing in strings and magic numbers (hardcoded values with no context) for each instance. This approach is fine for small programs, but it may introduce risks as the program grows (typos, inconsistent starting HP values, or incorrect names yet again).
> INFO:
> [Records](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) were introduced in C# 9.0.

## Thoughts Before You Go
Everything discussed here isn't exhaustive, and I'd be remiss if I didn't mention that the recommended solution above using computed properties isn't prescriptive, the only, or even "the best way". I didn't dive into some advanced topics such as a `Dictionary<>` which could be a great fit to this problem if you begin managing a larger set of data-oriented types.

That said, the `Monster` example used was purely demonstrative data-only class design. In a real game, monsters would certainly have *behavior*, and that's where inheritance and polymorphism would shine. Here, the goal was to focus on a narrow, introductory case for beginners where the only difference are static data values.

As you continue learning, remember to stay flexible. If you're stuck thinking "what if *this*?" or "what if *that*?", you may not make much progress getting any closer to a viable product. Code for what you need now. As the requirements change (they always do), you'll adjust accordingly.

I'll stop there, because going any further and we start getting into [YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it) territory, to which there's plenty out there to read about.

## Conclusion
This anti-pattern occurs when we create class hierarchies solely to represent different static values. The maintenance cost grows linearly with new types and exponentially with new properties. Just to hold data without different behavior.

**Beginner-Friendly Solution:** Try using a single class with computed properties (either with a getter or a separate method) derived from an enum. This centralizes logic, maintains type safety, and provides a single source of truth. For smaller projects, static factory methods can make things cleaner by avoiding larger constructor calls. Give each of them a chance and see how you like it. If you don't, try something else!

Remember: Use inheritance when subclasses have **different behavior**, not just different data.
