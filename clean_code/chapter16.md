## Chapter16: Refactoring Serial Date  

[*Original code*](https://github.com/jfree/jcommon/blob/master/src/main/java/org/jfree/date/SerialDate.java)  
[*Refactored code*](https://github.com/ludwiggj/CleanCode/blob/master/src/clean/code/chapter16/solution/DayDate.java)

### Abstract factory pattern (p. 273)  
**Definition**: A method to encapsulate a group of different factories with a common theme, but different implementation.  
**Use case**: 
* You want to create a Windows or Mac button, dependent on the OS used.  
* You have an abstract ButtonFactory class  
* You have WinButtonFactory and OSButtonFactory, which implement ButtonFactory 
* To create button, you evaluate a conditional (Mac or Windows), and then create *factory*, either an instance of either WinButtonFactory or OSButtonFactory
* You then call the create method on *factory*  

**Example**  
```
interface IButton
{
    void Paint();
}

interface IGUIFactory
{
    IButton CreateButton();
}

class WinFactory : IGUIFactory
{
    public IButton CreateButton()
    {
        return new WinButton();
    }
}

class OSXFactory : IGUIFactory
{
    public IButton CreateButton()
    {
        return new OSXButton();
    }
}

class WinButton : IButton
{
    public void Paint()
    {
        //Render a button in a Windows style
    }
}

class OSXButton : IButton
{
    public void Paint()
    {
        //Render a button in a Mac OS X style
    }
}

class Program
{
    static void Main()
    {
        var appearance = Settings.Appearance;

        IGUIFactory factory;
        switch (appearance)
        {
            case Appearance.Win:
                factory = new WinFactory();
                break;
            case Appearance.OSX:
                factory = new OSXFactory();
                break;
            default:
                throw new System.NotImplementedException();
        }

        var button = factory.CreateButton();
        button.Paint();
    }
}
```

### Enums (p. 275)  
**Use case**: You have a list of constants, like MONDAY = 0, TUESDAY = 1, WEDNESDAY = 2 ...  
**Problems**:  
  * Makes it (somewhat) awkward to change 
  * Difficult ot iterate over. Would have to put in a list first 
**Solution**: Use an enum class. Cane be iterated over.    
```  
public enum Weekdays {
    MONDAY = 0,
    TUESDAY = 1, 
    WEDNESDAY = 2,
    ...
 }
```
**Further use case** Enums can also be used to get rid of switch statements (p. 283).  
**Example**  
You have a list of different types of intervals (CLOSED, OPEN, LEFT, RIGHT) and you want to check whether a number is within a range given the type of interval.  
Old code with ugly switch  
``` 
int[] range
int num
isInRange(range, num) {
    if (interval == CLOSED) {
      if (num > range[0] && num < range[1]) return true
    }
    else if (interval == OPEN) {
      if (num >= range[0] && num <= range[1]) return true
    }
}
...
```  
New code  
``` 
enum INTERVAL {
    OPEN {
        public isIN(range, num) {
            if (num > range[0] && num < range[1]) return true
        }
    }
    ...
}

isInRange(range, int, INTERVAL) {
    return INTERVAL.isIN(range, int) // NO SWITCH!
}
```

### Feature envy (p. 278)  
* When a class contains a method (or other things) that rather belongs in another class  
* Can also happen within a class: Method that takes an instance of its own class as an argument (p. 281)  

### Temporary variable (p. 279)  
* A variable with a short lifetime, limited (local) scope. For example, only used within one method.  
* Explaining temporary variables: use temporary values to make your code more understandable  

**Example**  
Old code  
```
 public static SerialDate addMonths(final int months, 
                                       final SerialDate base) {

        final int yy = (12 * base.getYYYY() + base.getMonth() + months - 1) 
                       / 12;
        final int mm = (12 * base.getYYYY() + base.getMonth() + months - 1) 
                       % 12 + 1;
        final int dd = Math.min(
            base.getDayOfMonth(), SerialDate.lastDayOfMonth(mm, yy)
        );
        return SerialDate.createInstance(dd, mm, yy);

    }
```
Improved code:  
```
  public DayDate plusMonths(int months) {
    int thisMonthAsOrdinal = getMonth().toInt() - Month.JANUARY.toInt();
    int thisMonthAndYearAsOrdinal = 12 * getYear() + thisMonthAsOrdinal;
    int resultMonthAndYearAsOrdinal = thisMonthAndYearAsOrdinal + months;
    int resultYear = resultMonthAndYearAsOrdinal / 12;
    int resultMonthAsOrdinal = resultMonthAndYearAsOrdinal % 12 + Month.JANUARY.toInt();
    Month resultMonth = Month.fromInt(resultMonthAsOrdinal);
    int resultDay = correctLastDayOfMonth(getDayOfMonth(), resultMonth, resultYear);
    return DayDateFactory.makeDate(resultDay, resultMonth, resultYear);
  }
```

### Errata  
* Simplifying functions: If a function is only ever called by another function, think about integrating it into the other function (p. 277). *Seems like this could violate SRP*  
* Flags: Generally a bad idea to pass flags as arguments.  
* static/non-static: if a method calls on a class's instance variables it shouldn't be static  
* Abstract methods: If an abstract method does not depend on anything in the implementation of that class, it doesn't need to be abstract. It's implementation can be written in the abstract class itself. (p. 282)  
* Logical dependencies should be physical dependencies: If class1 makes an assumption about properties of class2, it is logically dependent on class2. This should be implemented in code (physical dependency), e.g., by class1 retrieving this property from class2. (p. 282, 298)  

