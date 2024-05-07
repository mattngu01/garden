> injection = passing by reference the object that is used by the class / function / thing

https://github.com/iluwatar/java-design-patterns/tree/master/dependency-injection
## Example:
- a wizard wants to be able to smoke different types of tobacco

## What it's useful for
- unit testing & mocking
	- would be hard to test a function that does something to a request, when it makes a REST `Session / Connection` object within that function. Ideally we would want to have that passed in, so we could mock the `Session` object to return our ideal response
- decoupling behavior? If this does not happen, violates [[Single Responsibliity]] 

## Code Example
```java
public abstract class Tobacco {

  public void smoke(Wizard wizard) {
    LOGGER.info("{} smoking {}", wizard.getClass().getSimpleName(),
        this.getClass().getSimpleName());
  }
}

public class SecondBreakfastTobacco extends Tobacco {
}

public class RivendellTobacco extends Tobacco {
}

public class OldTobyTobacco extends Tobacco {
}

public interface Wizard {

  void smoke();
}

public class AdvancedWizard implements Wizard {

  private final Tobacco tobacco;

  public AdvancedWizard(Tobacco tobacco) {
    this.tobacco = tobacco;
  }

  @Override
  public void smoke() {
    tobacco.smoke(this);
  }
}


var advancedWizard = new AdvancedWizard(new SecondBreakfastTobacco());
advancedWizard.smoke();

```