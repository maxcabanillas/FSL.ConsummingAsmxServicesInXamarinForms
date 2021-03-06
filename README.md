# FSL.ConsummingAsmxServicesInXamarinForms

**Consumming ASMX Services in Xamarin Forms**

When I started work with Xamarin Forms one of the first things I needed was consumming services. When you use Xamarin Forms consumming RestFull API services is quite easy (I will post that later). But, ASMX services will need some additional implementation for different plataforms.

![enter image description here](https://fabiosilvalima.net/wp-content/uploads/2017/01/fabiosilvalima-consumindo-servicos-asmx-com-xamarin-forms.jpg)

> **FULL ARTICLE:**
>
> English: https://fabiosilvalima.net/en/consumming-asmx-services-xamarin-forms/
>
> Português: https://fabiosilvalima.net/servicos-asmx-com-xamarin-forms/

---

What is in the source code?
---

#### <i class="icon-file"></i> FSL.ConsummingAsmxServicesInXamarinForms

- An interface for the service with all methods we want. It will be at portable project.
- The Service Web Reference. References the service in both Xamarin.iOS and Xamarin.Droid projects.
- A class that implemments the interface created above. That class will be wrapper for the service itself. It must be in both Xamarin.Droid and Xamarin.iOS projects. So, yes, you will need to create two classes.
- Add a Xamarin Forms dependency (dependency injection) for that interface in both Xamarin.Droid and Xamarin.iOS projects.
- A static property in App portable project to consume the service. That property will be "injected" by Xamarin Forms.

> **Remarks:**

> - First of all, I have created a cross-plataform Xamarin Forms portable project using Visual Studio 2015. For the article, I don't use any third party component to manage services. I do like to create my own implementation for understanding purpose. 

---

What is the goal?
---

- Consumming a ASMX web service in Anroid and iOS projects;

**Assumptions:**
- You need to create a virtual directory in your IIS for the web service.


Explaining...
---

Let's say you have a view and you want to get all customers from your ASMX service. You have a method that receives a criteria filter and returns a list of customers.

First, reference that service in Xamarin.Droid project.

Second, create an interface "ICustomerSoapService" and a GetAllCustomers method:

**Models**
```csharp
public interface ICustomer
{
        string Name {get;set;}
        string Id {get;set;}
}

public interface ICustomerSoapService
{
        Task<List<ICustomer>> GetAllCustomers(string criteria = null);
}
```

Ok, now create a class "CustomerSoapService" in Xamarin.Droid project that implemments that interface "ICustomerSoapService":

**Models/CustomerSoapService.cs**
```csharp
[assembly: Dependency(typeof(FSL.XF2.Droid.CustomerSoapService))]

namespace FSL.XF2.Droid
{
    public sealed class CustomerSoapService : ICustomerSoapService
    {
        CustomersWs.Customers service;

        public CustomerSoapService()
        {
            service = new CustomersWs.Customers()
            {
                Url = "http://localhost/FSL.WS/Customers.asmx"
            };
        }

        public async Task<List<ICustomer>> GetAllCustomers(string criteria = null)
        {
            return await Task.Run(() =>
            {
                var result = service.GetAllCustomers();

                return new List<ICustomer>(result);
            });
        }
    }
}
```

The "CustomersWs" code above it's your web service reference.

In your App portable project, create a property to be inject (dependency injection) by Xamarin Forms.:

**App.xaml.cs**
```csharp
public partial class App : Application
{
        private static ICustomerSoapService _customerSoapService;
        public static ICustomerSoapService CustomerSoapService
        {
            get
            {
                if (_customerSoapService== null)
                {
                    _customerSoapService = DependencyService.Get<ICustomerSoapService>();
                }

                return _customerSoapService;
            }
        }
}
```

The property above is lazy loader, ti will be injected by Xamarin Forms only if someone uses it.

Finally you will call the method you want in your view:

**MainPage.xaml.cs**
```csharp
var customers = await App.CustomerSoapService.GetAllCustomers();
```


----------

References:
---

- Install and Config Xamarin Forms [click here][1];
- More at my blog [click here][2];

Licence:
---

- Licence MIT

---

![Programação no Mundo Real Design Patterns Vol. 1](https://www.fabiosilvalima.net/wp-content/uploads/2017/02/fabiosilvalima-ebook-design-patterns-INSTAGRAM-2.png)

https://www.fabiosilvalima.net/ebooks/design-patterns-vol-1/


  [1]: https://fabiosilvalima.net/configuracao-xamarin-visual-studio-2015/
  [2]: https://www.fabiosilvalima.net
