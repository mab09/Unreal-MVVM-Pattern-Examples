# Unreal MVVM Pattern Examples
This is a project where I tested out and learned how to use the [UMG ViewModel Plugin](https://dev.epicgames.com/documentation/en-us/unreal-engine/umg-viewmodel-for-unreal-engine), through this I was able to make a MVVM-based UI architecture for one of the larger projects I was working on which I will touch upon at the end.<br />
> [!IMPORTANT]
> ViewModels in Unreal is a beta feature with not alot of documentation and examples out there showing the different ways you can set it up.<br />
> In this project I have tried to tackle one of the areas that I had the most trouble with initially which was how to link your widgets/views to your viewmodels.<br />
> Hopefully this can help whoever is trying to setup their UI with MVVM and show the possible approaches they can take.

That said, there are some great resources other than Unreal's own documentation that helped me understand the Hows and the Whys when implementing this. I won't be going into the basics of what MVVM is and its advantages over traditional MVC, so I recommend going through the links below.
- [Building More Robust and Testable UIs using MVVM | Unreal Fest 2023](https://dev.epicgames.com/community/learning/talks-and-demos/pw3Y/unreal-engine-umg-viewmodels-building-more-robust-and-testable-uis-using-mvvm-unreal-fest-2023)
- [Model View ViewModel for Game Devs](https://miltoncandelero.github.io/unreal-viewmodel)
- [Handling UI Navigation with MVVM and Common Activatable Widgets](https://dev.epicgames.com/community/learning/tutorials/ep4k/unreal-engine-handling-ui-navigation-with-mvvm-and-common-activatable-widgets)

## Project Information
- This project uses the ThirdPerson template and implements some basic UI elements:
  - **Health Bar:** Showing health from a _AC_Vitals_ component on the character (A debug key 'H' that reduces the health to show the UI updating)
  - **Stamina Bar:** Showing stamina from _AC_Vitals_ (The player can sprint while pressing 'Shift' causing the stamina to deplete and then regenerate when released)
  - **Pause Menu:** A pause menu with options to resume or quit the game (press 'esc')
  - **Map:** A mini map and a full screen map that can be cycled through (press 'M')
- The main assets to look at are in [Content/ThirdPerson/Blueprints](https://github.com/mab09/Unreal-MVVM-Pattern-Examples/tree/main/Content/ThirdPerson/Blueprints) and [Content/UI](https://github.com/mab09/Unreal-MVVM-Pattern-Examples/tree/main/Content/UI)
- This project is all blueprints, since the goal is to show how to link a viewmodel to a view/widget which are usually blueprints. You can have the ViewModels be C++ classes instead, how they are accessed by the widgets remains the same.
- Made in Unreal Engine 5.6.1
> [!NOTE]
> This project is not an example of best UI/UMG practices or what an ideal architecture should be, the goal is to showcase the different ways viewmodels can be setup to work with your widgets.<br />

## The Model, The View and The ViewModel
Here is a simple diagram showing how the data flows between the different UI elements in this project as per the MVVM design. <br /> 

<img width=75% height=75% src="https://github.com/user-attachments/assets/b5fdd078-f54b-40cc-8c74-d9b8306934d7" />

## Who Creates the ViewModels and How it Connects to the Widgets?
This is the hard part of setting up the MVVM UI in your project, not because it is complicated but because there are multiple ways you can do it (called the "Creation Type" when adding a viewmodel in the widget), and each of these require different setup.<br /> 

<img width=50% height=50% src="https://github.com/user-attachments/assets/fabb298c-ed26-4ae8-81c4-99f12e96ea37" /><br />

For the most part in this project, all the widgets and the viewmodels are created and initialized in the [HUD_MVVM](https://github.com/mab09/Unreal-MVVM-Pattern-Examples/blob/main/Content/ThirdPerson/Blueprints/HUD_MVVM.uasset) (Check the `CreateWidgets()` and `InitializeVMs()` functions), how it all comes together depends on the creation type.
> [!TIP]
> The HUD in my opinion is a good place to create and initialize these UI related objects, another way is to have something like a UI manager component on the player controller or player state.

### Creation Type 1: Manual
> The widget initializes with the Viewmodel as null, and you need to manually create an instance and assign it.

This is the most straightforward way to link your viewmodel to a widget. There is no active example of this one in the project but the screenshots below should be enough to show how to implement.<br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/9fe9eab2-3890-4692-a6e9-4c687837bc0e" /><br />

Since this creation type always creates a public setter for the viewmodel in the widget therefore the reference to the viewmodel can be passed in at the time of widget creation. Just make sure your viewmodel is created and initialized before widget creation.<br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/7ff4ea0b-2ad9-4999-9b82-add23b57c25b" />

> [!TIP]
> Since the widget links to a viewmodel before `Construct()` it is recommended that your viewmodels are created and ideally initialized before the respective widget is created.

### Creation Type 2: Create Instance
> The widget automatically creates its own instance of the Viewmodel.

The [Health Bar UI](https://github.com/mab09/Unreal-MVVM-Pattern-Examples/blob/main/Content/UI/Widgets/WBP_StatusBars.uasset) is implemented through this type.<br />
Since the ViewModel is created locally in the widget we must also initialize it here with the model (_AC_Vitals_) <br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/dd3444f3-8b9a-454e-933f-0f637a948c86" />

> [!NOTE]
> This is a pretty easy way to setup the viewmodel but it is questionable with respect to MVVM decoupling since the View/Widget itself has a Model reference


### Creation Type 3: Global ViewModel Collection
> Refers to a globally-available Viewmodel that can be used by any widget in your project. Requires a Global Viewmodel Identifier.

The [Map UI](https://github.com/mab09/Unreal-MVVM-Pattern-Examples/blob/main/Content/UI/Widgets/WBP_MiniMap.uasset) is implemented through this type.<br />
In this method, after initialization, the viewmodel is stored in a global collection of viewmodels with a unique identifier. This identifier is later used by the widget looking to find this viewmodel.<br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/d88bd7f4-ea15-453a-a8c2-14242f41a346" /><br /><br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/faaf0feb-9475-44fe-9cca-ddae1f653948" />

> [!TIP]
> Through this method, your widget blueprint is completely free of any model related information, allowing a highly decoupled architecture.<br />
> In a larger project this might become harder to maintain considering the number of viewmodels that might exist, in that case, this method can be used for parent viewmodels that encapsulate a bunch of other small ones, which in turn may use a different creation type like 'Resolver'

### Creation Type 4: Property Path
> At initialization, execute a function to find the Viewmodel. The Viewmodel Property Path uses member names separated by periods. For example: `GetPlayerController.Vehicle.ViewModel` Property paths are always relative to the widget.

The [Pause Menu UI](https://github.com/mab09/Unreal-MVVM-Pattern-Examples/blob/main/Content/UI/Widgets/WBP_PauseMenu.uasset) is implemented through this type.<br />
In this method, the viewmodel may be initialized anywhere as long as the widget can find its reference through a chain of functions/properties. Here  we initialize the viewmodel in the HUD and then the widget finds it through a local helper function `GetViewModel()`<br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/e00d9fa0-6363-4452-8a0c-9228ea3cd36c" /><br /><br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/7a1c1e31-971b-4b11-aebc-0d5edac2f2d1" /><br /><br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/15ec5259-2ef4-44cf-bea9-357c9d79106f" />

> [!TIP]
> - **This function must be pure and const**<br />
> - We need not have the viewmodel finding logic in the widget, it could be in any blueprint as long as the widget has access to it.
> We could techinically use the path "GetOwningPlayer.GetHUD.pauseMenuVm" but we can't cast to our custom HUD in the property path and this has to be a design time resolution.


### Creation Type 5: Resolver
The [Stamina Bar UI](https://github.com/mab09/Unreal-MVVM-Pattern-Examples/blob/main/Content/UI/Widgets/WBP_StatusBars.uasset) is implemented through this type.<br />
In this method, each ViewModel comes with a ViewModel Resolver (VMR), The resolver's sole purpose is to locate the ViewModel. <br />
Here the stamina ViewModel is intialized in the player HUD and then the resolver finds that viewmodel using a blueprint interface implemented on the HUD.<br />
<img width=50% height=50% src="https://github.com/user-attachments/assets/5a10bbe3-3097-438e-afde-a3bcea39fdb2" /><br /><br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/a5dcc21f-4965-4bea-87e3-cc61e495900e" /><br /><br />

<img width=50% height=50% src="https://github.com/user-attachments/assets/1189b785-39e1-4291-b2e5-5257a52ae318" />

## Closing Thoughts

https://github.com/user-attachments/assets/b2fa917b-367e-472b-8087-2bda625ebaa6

As I mentioned before, this project looks at a very specific aspect of setting up MVVM in unreal, which seemed to have been a common point of confusion and difficulty (atleast at the time I made this), So I hope this project covers that aspect extensively enough to clear the confusion.<br />
There are other aspects like how to setup the bindings which I won't be writing about here but you can go through the widgets to have a look.<br /> 

Overall here are some of the things you might want to consider when setting up your MVVM architecture:
- The ViewModel to Widget relationship can be one-to-many or many-to-one.
  - Many-to-One is useful for stuff like the StatusBar widget where one widget needs information of different things like the Health and Stamina.
  - One-to-Many is useful if one piece of information might be needed by different widgets like health is needed by the health bar and maybe also a screen effect UI.
- Property bindings can be _One Time to Widget_, _One Way To Widget_, _One Way to ViewModel_ or _Two Way_ , out of which I was not able to fully utilize _Two Way_ in this project, that should be useful for highly interactive  widgets like an inventory menu.
- Property bindings have in-built conversion functions, so you can do basic math, string conversion and much more in the binding itself without having to write custom logic for it.
- I believe that the 'Global ViewModel Collection' and 'Resolver' creation types give the maximum decoupling, a combination of the two might be the best approach. But note that achieving the maximum decoupling might be overkill for small projects and smaller teams. In my opinion, this is mostly beneficial if you have different people working on the UI functionality vs the UI visuals.
- An MVVM style architecture itself may be overkill for a project. While it is possible to base your entire UI on MVVM, it might be more beneficial to use it in certain UI elements, while sticking to traditional ways in others. Remember, this is still a beta feature so basing everything around it may restrict the usage of other valueable features like CommonUI.

## Example of an Advanced Architecture
Here is one of early iterations of the UI architecture I implemented in a larger project that I was working on. We made quite a few changes later on but this is the first working example I made of a fully MVVM based UI architecture in Unreal. <br />
This involved nested viewmodels where the master viewmodel and the viewmodels for different UI layers were setup using the global viewmodel collection while the viewmodels for the individual UI elements were setup using resolvers<br />

[Advanced MVVM Architecture](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=MVVMArchitectureExampleForUnreal&dark=auto#R%3Cmxfile%20pages%3D%222%22%3E%3Cdiagram%20id%3D%22o6ScfwJmkazL59GZc312%22%20name%3D%22Page-2%22%3E7V3rk6K4Fv9rumrmg10kvD%2B29mOmanpv1%2FSsc%2B9%2BmYpCKzsIvYD92L%2F%2BJkAUSES0QaJmq3fUGEk4Oed3Ts4jXKijxdtdhJ7n96Hj%2BhdQcd4u1OsLCIFt6PiFtLznLYpqZC2zyHOyNmXd8Oj969KOeevSc9w4b8uakjD0E%2B%2B53DgNg8CdJqU2FEXha7nbU%2Bg7pYZnNHNL0yANj1Pku0y3n56TzLNWSy%2F0%2FuJ6szkdGSj5NwtEO%2BcN8Rw54WuhSb25UEdRGCbZu8XbyPUJ9cp0ud3w7WpikRskTX7ww1t8%2BetmBIzn8ddxOLv7a%2Fw6HlhmPrnknd6x62AC5B%2FDKJmHszBA%2Fs26dRiFy8BxyWUV%2FGnd51sYPuNGgBv%2FdpPkPV9NtExC3DRPFn7%2BLZ5x9P5f8vtLnX78X3659MP1W%2BnTe%2F6JveWcCnG4jKb5%2FL1lGP14HU7m0e9%2FDPT1j4lzczeAOeugaOYmNfQAOXcSIhRGyCl654YLF08Id4hcHyXeS5lLUM5ss1W%2F9XrgN%2FmS8JenbtovyF%2FmI3358%2FrX%2FXh8fwENH9%2FI0PFeSstn%2FLMMs%2Fb42UeYcFdBGLjr9vWItOXxeuSHS%2BcaJajYzZjlr372epuORFsrPBO%2FegsfBYQ5nsIgoexD1hr53izA76d4zdwIN7y4UeJhAbvKv0gI0wync893vqH3cEmWJ07Q9Df9NJyHkfcvviyi%2FIO%2FjpKcu6BR6vFIfpmzS%2BTGuM8DZRdQabpHb6WO31Cc5A3T0PfRc%2BxNVrexwJzjBcMwScJF3ul17iXu4zNKGe8V41%2BZyTcyKyGA%2B1bLXfm3KmXHHD0HUMnx9HWNRRhls7Z5EYcg3MyS%2BXjfMV6iYIZvceOAmskMtwK44nCqUh4N%2BXitA5S4QwIVMSMHq1vdXzRURjQor04om%2F54f8ZfXRV4eFLga7QgCxZMYvJyheUKX%2B0%2Fk7%2BJEhFKtjCvJCnPR%2BFvdxT6IRai63RgLGye71eaqLz57lOyUdpizLVeMPuW9rnW1i3f8zUlTSH%2B7ZOfqqy55zhukEJ%2Fgu8lEwsiA8%2BhFyTpoutD%2FIfZYEQwXccTH%2BHPYP0Z%2F5HuUTIKA3wvyEulwsUy9%2BoSudtdnGoRc7uM5SwOjWYSRfu1jvEah5ErLOB76dJmLEDNELDX%2Bi%2FwSqZKPF%2FwH6nKHgCGKVSWKVQOA%2Fho4voPYewlXkiuH2V9K4yxbe3LzO0Fczfykg4XXYfNFt3qaM317eB1uwymhKbxZgTLfhDRFnCJLzmKXAy7ZM6Bk5opMy8mehcq4%2Ftf9yh7X%2F0lLP3y5%2FCh0lMC4RkAoaE0kwmjxrJoKhRc25s1dhmW7YkRRZICaWp3YmoPAAV7amsD45JjbEOO9QtNfR9je4Dt5sqQnBF7sLe50vlRe1vBww7TP2XqoxjPUflTyrfUco1FulZnfMjc58p0C%2BY%2Bd8bS3N%2FF3G9n0XnmPm%2FRWzD3uRPebO5zYGjnHUAZUL4GeG0wVxAl%2B%2B5G4%2Fv402fW4r9zkzu0yLrUfH%2FvBsu6Plu%2FDx3kb%2BzwPd%2Bc1E1UagapGQ6jGXj7Hx5IANCCt58fjJGq4eCqAZigX91g76UbxijyCO130w3E4E7Blgy4WCkAFnXZjiuk39a5eceVbpAYLzH%2BMBgPTLuZtNtdmf80r6Hs5Tp7MRBJBqWHrRsPG1S0krtL1a2Ge3HV3se%2FJmYwm48KgEGF3bxr5Wg239UmQUYgkJGKfhdFv9Kax%2BPnA2w4S%2B7mdtzN7b7sfXv6wOYwSeeuPuI%2BUwb4%2F2WwjF2nuP%2B6LfjkvsZp76spSV8su9xW3R6bdbtynBExdcb3Gzrg4eo7fHcXGJCYPueoIKR2Oh%2Ft1L%2BvEcg4VA%2FqqXdvI9gvFLWfu3EShmRCk5Iq4Qz34x4LHt6rZFogPmccFmBwqRp6VQ2NXZSG0hFGaBaz6AeoCdroYWqvFCf%2F6QNhkrWbSIP6pW4CDVrZv3bJaQT0iu8pKzXKr7GmNHNZS694n8gw5Utl1UjMpa6iCL0XuuVMXXMD5ZGgre80s0p%2F%2FCabQaseLs3ug6laLzTrmRkNbT9mBEDXKPdRflQVQfiRTO7wDEmdwwKmG8t4RB%2FFdZrK6l1e%2Fjt1MX0sGAFU1lMlTDTC3LhJ2CcaURArWWB3%2FBat0VjQeqiw46fYsK6NPg3aYrE6P60hF%2Fmt1erQbtsq%2BVgBDze3gWQPDVHsCrG5lYNLW%2BYEbZmBXtnTmbrJq14yLBZ2AT0D5GSzK%2BAhsiskzImFNNKq2ymTspUECx68dGbV0YCVjGB9IF2%2BlQQL3rJ3FcCCh0ywqLkyXuGAi3ITbFbM0s3DYJotyxUx5SMUxHQ5UtWx%2Fs4nVBs4KPr9KZpNPhEwwK305XP2Sr6Bup59KL75%2FDmbVQUl17khWV5FmhgyxDR7jjAT%2FZHuBm5eyHSYNP1HN8GignceYw8bNp7v4U0O5l13dRnc4YIeMIH5GjlTYhDx6r%2Fqr3Prub7zR5h4T%2B8F%2BmeUFVjRyMGlfj99%2Fc5LUeEBval1hfQyQ6UH%2FQ5UrWcFf8gMlZtHHyuktX7K7ptVWZsVVv9ILAQ4n%2FvgUln1qqyA1nBbotWc1PMx2GLDycOHX%2BP7XwQ%2FlvEQ9R1UloOf3%2BDS%2BX6YwkZqmVLnu6Y2db43SRE6cud7u8kEw4e13514QBxsEL%2BQYhflKcJrnh5zKV3x5zy4NL0%2BYnq1kt1x2DiAPDbn436CnZe99zjAfufmtBYH2FR7SaQ%2B8oi0Uw95VoupPIUk%2F22RPyFl4gUOZoxVrUu1avLBjYgpNiZ3V%2Bcvr1ZtjpZRxPxuq7%2B%2BVB46DYO196NubKE23AJMSQ4uNf0pa%2FqmEQG7Kx%2BLyuYXSk3fuaYHesO9bFeqXt2cvtV%2BRADLMSLaclpQpFwVWNqVcuZDL7RAb5svcrbQLeMXYg0uoxoCKlxgcEqSeMird3Zwtcos%2BjFWbW4tpNByy6Lr8k6oKZer2k78b7VG2LSrNZmNy41BuYbSNLop7oSUVNQozHm06bx27W%2FlZkWntaC0Fo8tXBHhzDwZxTlEFAdQM3NVr2%2FxgjjnWRGqtvu0xcI50LIg9NjtmhV4Hk9BqMqmFuZ4z5xeLgZHSszvBPM1rXImhsT8gpA0eEilxPwzxXytsagJg%2Fn8814k3p8R3ptaZb8t8b4gIK2f%2BiLx%2FmTw%2FvgOfVHZtCCC96I8dEgC%2FkGcOoq08GtEZHMOlUT8c0d8q7GsiYL4GpseglnyyzKYSbQ%2FC7Q3lTL20tiRhPqB1u4RSExpm0T8o0d8beeqy%2F4RHzJcTRD%2F3YlQmmMnFDdK0O%2FGhw8l6G8Sj3ajthL0TxD0YWN5EwX0TZNZ8wPkpDkonqe%2Fpx8eUEIEOG2BCqwDs60H%2Fmp5OGVrnhp9GIUgB%2F5q3Dj6Fxdj21wsVJDKt6OkKSiV7wbZaDd8LpXvCSrf44uia70o39YTwrdqZPoIjK0amWZCtJw53vUzd3Rdr%2BvfTZq1Dvpgnv2ZgGa9bi8fEIMJqrn5uglrmWBL%2F46YAJ4qE0AhmWBbwYWu1hZodMQEvLM9K8ZRdnbwBtuoWj74E9tH5Wd3bbFUCrzBU%2BYt2MbV5FKbjTsDmnNTOlC1q%2Bc66g0s0p2JzhTJ9E53WHmanGYpnJD%2FgUnPJh%2B2Q3omf7F38uuGgORvkNq2O9wIR3rLAuKRnnc604dJz2Rx9U57AAxdPOI3SPDZmfiZe69woGn%2FtDfViqbluIUOSniDTUVpgfA0rikS7bWKB1AA2jdIddiZ9pnjjZyNFQtDegMKx%2FawC7ZPE7hE4nlLrVr2fRPe7OWR7d3GxJruuynW9rzv1tWyB84GoLisu%2FbvZt9tKX3wyb7u24%2FwV%2BcPardXD1iksS%2BrIt1Nj%2B4wdPVSV9b%2FbQCXlg%2FyqPIfnX5zfrW23JVW178j%2Fu7Fwywyn1qwyqUVdmrKpbZStTYqD%2F5piS%2BrE97GZ8y8DsJnfZ%2BLZJ4MkLbGoCbUCVlE5FET1mJnRzzK%2Bthpec4394WcQlzh4HiOSDrC9XLh30Zo4W4woAsMmdLJjdLnCMYNeI0MCGq5bZenLljlQIdKHxtTtMINJb9wKRsEdHVCqdWLHS5yeBx2giK7h8drw92cIBqo69%2BRwLLeOhrSOhGJVcpUXR3dX5BYi8pmOX2rqwMObdZZNCKJfqdCcr0CkqbC8UkbtBC0RPPqAYKt0Rwo8CRQsmvLCABLu9ShrQFgGhpeObssPUCx97OULMuo7F0HhlI9eLIlUwnfRGUoqx56Lcus698N9AKlF4N%2BfwVsN%2FWO0awDQTLGgaKdKqENwQitnyqhzU7gt3NgA5rRA7LZDWrAjj3TCoBKXQDen9ismXPQiAygtehHI39Na5BsSzCg6yUl%2FRCEbr3Yaz9nV8Ug0%2BrjWng9QE3%2Fruy3XhwvB2CCVTnTkXEB1KweuMA%2BWS7oZs%2FZNRdo8PBcYLN%2B71NMc7ZAmbSFLXxvRo%2FdVYa5eLm2QIEbXSj90b%2BLNHMBaQ%2BpF0Ek2neSYy5gorNqCwg8XWSZi5foXMX83ne5dIty%2BnnO2A6HYtF%2BtTM48WxbsHLmiEP5c0kxB6ouGu0By%2FUZqWOGPu3Ealsgo2mUo7ADyAnDWjw6qjQpYAdC4o9RSE7vWO%2Bq8I3O79NHj6s3%2Fwc%3D%3C%2Fdiagram%3E%3C%2Fmxfile%3E#%7B%22pageId%22%3A%22o6ScfwJmkazL59GZc312%22%7D)
<img width=100% height=100%  src="https://github.com/user-attachments/assets/6d11c8cb-8c25-461a-820c-7ef46c8a8aed" />
