---
layout: post
title: Expandable ListView in Xamarin Forms
date: 2016-12-11
comments: true
categories:
- Mobile
tags:
- Xamarin
---

There's no out of the box functionality to create expandable list views in Xamarin Forms. A quick search on Google returns quite a few examples of how to achieve this using custom renderers. 

Another easy way of doing this is to enable grouping in your ListView and bind to a property which contains key, value pairs of. You can then add a check for a “Tapped” event on the Category to either clear or populate its list of items.

<!-- more -->

Example code is available on my GitHub. The example uses Prism to handle the MVVM bindings. [https://github.com/umarmohammed/XFExpandableListView](https://github.com/umarmohammed/XFExpandableListView)

## ListView Grouping Headers
Grouping headers are a way of grouping items in a ListView by category. This blog post by James Montemagno provides a good tutorial:

[http://motzcod.es/post/94643411707/enhancing-xamarinforms-listview-with-grouping](http://motzcod.es/post/94643411707/enhancing-xamarinforms-listview-with-grouping)

## The Data

Lets start out with the following data models:

```csharp
public class Item
{
    public int ItemId { get; set; }
    public string ItemTitle { get; set; }
 
    public Category Category { get; set; }
}
 
public class Category
{
    public int CategoryId { get; set; }
    public string CategoryTitle { get; set; }
}
```

We have a list of items that we wish to display, and each Item has an associated category. In our example our data will be of the form:

```
{
  "data": [
    {
      "category": {
        "categoryId": 1,
        "categoryTitle": "Category 1"
      },
      "itemId": 1,
      "itemTitle": "Item 1",
    },
    {
      "category": {
        "categoryId": 1,
        "categoryTitle": "Category 1"
      },
      "itemId": 2,
      "itemTitle": "Item 2",
    },
    {
      "category": {
        "categoryId": 2,
        "categoryTitle": "Category 2"
      },
      "itemId": 3,
      "itemTitle": "Item 3",
    },
    {
      "category": {
        "categoryId": 2,
        "categoryTitle": "Category 2"
      },
      "itemId": 4,
      "itemTitle": "Item 4",
    },
  ],
}
```

We have four Item objects, two of which belong to category 1, and two to category 2. We want to display them in an expandable list view where: initially the categories are displayed, when tapped the category expands to display the items.

<div>
<img src="/images/screenshot1_cropped.png" />
</div>


## Structure the Data

Currently our data is simply a list of Items. We need a way of representing that data so that we can group it by category. We also need to signify whether or not category should be expanded.

We do this by creating the following classes:

#### Category ViewModel

We create a new class to wrap our Category model. This ViewModel contains an extra boolean field to signify whether or not the Category has been selected

```csharp
public class SelectCategoryViewModel
{
    public Category Category { get; set; }
    public bool Selected { get; set; }
}
```

We will later use the Selected Boolean field to decide if we should expand or contract the list.

#### Create a Grouping Data Structure

We need to create a grouping structure to store our data. This structure allows us to restructure our data into a list of key value pairs.

```csharp
public class Grouping<TK, T> : ObservableCollection
{
    public TK Key { get; private set; }
 
    public Grouping(TK key, IEnumerable items)
    {
        Key = key;
        foreach (var item in items)
            this.Items.Add(item);
    }
}
```

For a more detailed explanation of this structure and grouping data in a ListView read this blogpost:

[http://motzcod.es/post/94643411707/enhancing-xamarinforms-listview-with-grouping](http://motzcod.es/post/94643411707/enhancing-xamarinforms-listview-with-grouping)

## The Page ViewModel

#### Initialise the Collection

We bind our ListView to an Observable Collection of Groupings called Categories

```csharp
public ObservableCollection<Grouping<SelectCategoryViewModel, Item>> Categories { get; set; }
```

Categories is initialised to contain two Groupings; one for each Category. The Key is set to a new SelectCategoryViewModel which wraps the Category and the Items are set to a new empty list.

```csharp
Categories = new ObservableCollection<Grouping<SelectCategoryViewModel, Item>>();
var selectCategories =
  Data.DataFactory.DataItems
    .Select(x => new SelectCategoryViewModel 
    {
      Category = x.Category, 
      Selected = false
    })
    .GroupBy(sc => new {sc.Category.CategoryId})
    .Select(g => g.First())
    .ToList();
selectCategories
  .ForEach(sc => 
    Categories
      .Add(new Grouping<SelectCategoryViewModel, Item>(sc, new List<Item>())));

```

The code example above:

1. Initialises Categories to new ObservableCollection.
2. Creates a list of SelectCategoryViewModels from the data by searching for unique Category objects in the list of data items.
3. For each SelectCategoryViewModel adds it to the Categories property.

#### Expanding/Contracting the List Of Items

When the Category is pressed we need a Command to either populate the list of Items with the data for that Category, or to clear it.

```csharp
public DelegateCommand<Grouping<SelectCategoryViewModel, Item>> HeaderSelectedCommand
{
  get
  {
    return new DelegateCommand<Grouping<SelectCategoryViewModel, Item>>(
    g =>
    {
        if (g == null) return;
        g.Key.Selected = !g.Key.Selected;
        if (g.Key.Selected)
        {
            Data.DataFactory.DataItems
              .Where(i => 
                (i.Category.CategoryId == g.Key.Category.CategoryId))
              .ForEach(g.Add);
        }
        else
        {
            g.Clear();
        }});
    }
}
```

Note: DelegateCommand is from the Prism Library

The Command above first flips the Selected field. It then checks the Selected field

* If True -> Populates the list of Items with Items of that Category
* If False -> Clears the list of Items

## The View

The data is displayed using a ListView with the “IsGroupingEnabled” property set to “True” and its “ItemsSource” property bound to Categories. The full code for the ListView is

```xml
<ListView IsGroupingEnabled="True" ItemsSource="{Binding Categories}">
  <ListView.GroupHeaderTemplate>
    <DataTemplate>
      <ViewCell>
        <ContentView Padding="10,0,0,0">
          <Label Text="{Binding Key.Category.CategoryTitle}" 
                 VerticalOptions="Center"/>
          <ContentView.GestureRecognizers>
            <TapGestureRecognizer 
            Command="{Binding Source={x:Reference TheMainPage}, 
                    Path=BindingContext.HeaderSelectedCommand}"                                      
            CommandParameter="{Binding .}"/>
          </ContentView.GestureRecognizers>
        </ContentView>
      </ViewCell>
    </DataTemplate>
  </ListView.GroupHeaderTemplate>
  <ListView.ItemTemplate>
    <DataTemplate>
      <TextCell Text="{Binding ItemTitle}"/>
    </DataTemplate>
  </ListView.ItemTemplate>
</ListView>
```
Taking a look at the above code in more detail:

#### The Categories Display:

The displaying of categories is handled in the GroupHeaderTemplate. The binding context for GroupHeaderTemplate is a Grouping. The DataTemplate is set to a custom ViewCell containing a Label which is bound the CategoryTitle. We wrap the Label in a ContentView so that we can add a TapGestureRecognizer to it.

Notice in the TapGestureRecognizer

```xml
<TapGestureRecognizer 
Command="{Binding Source={x:Reference TheMainPage}, 
  Path=BindingContext.HeaderSelectedCommand}"
CommandParameter="{Binding .}"/>
```

The Binding Context here is a Grouping, however we want to bind the Command to HeaderSelectedCommand in the Page’s ViewModel. We achieve this by setting the x:Name property of the Page to “TheMainPage”. We then set the Source and Path of the Binding accordingly. The CommandParameter is set to the current Binding Context signified by the “.”.

#### The Items Display

The displaying of Items is handled in the ItemTemplate of the ListView. We simply set to this to a TextCell with the "Text" property bound the "ItemTitle" of the Item.

So there you have it, you can create an expandable ListView in Xamarin Forms without much code using Grouping in a ListView. A full example is available on my GitHub account.

[https://github.com/umarmohammed/XFExpandableListView](https://github.com/umarmohammed/XFExpandableListView)


<div>
<img src="/images/listviewscreenshot.gif" />
</div>
