#### Encapsulation
Encapsulation is a principle that seeks to hide the implementation details of objects from the outside world. It states that all important information is contained within the object; only selected data is available externally. Each object's inner workings and state are stored privately within the specified class, whereas other objects do not have access to it or the ability to make changes. Instead, they can only interact with a few public functions or methods. This form of data hiding provides program security and control over object state changes, reduces the risk of errors, and makes the program more understandable.

```php
class Transaction  
{  
    public float $amount;  
  
  public function __construct(float $amount)  
  {  
    $this->amount = $amount;  
  }  
  
  public function process(): void  
  {  
      echo 'Processing $' . $this->amount . ' transaction';  
  }  
  
}
```

Setting the `$amount` to public is detrimental, because it means that the state of the application can be altered at any time. You could solve this issue by:

- [ ] Setting the access property of the amount to private.
- [ ] Using getters and setters
- [ ] Or just making sure that you only init the object from the constructor. That way, you ensure that only that instance of the object is running.

#### Abstraction
This simply entails that the internal details of a class implementation is hidden from the user.

#### Inheritance
This allows you to have a class that is derived from another class. This is also referred to as Parent Child class. The child class can inherit the public and and protected methods and variables.