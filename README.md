# Usage of iterator design pattern in the complex app

![Image of Airport](https://cdn-images-1.medium.com/max/800/1*06PLA1vPNmggQCKOGbnUoQ.jpeg)

Let’s talk about the widely used design pattern called ‘iterator’. Usually we can see the next description of it:
>*Iterator provides a way to access the elements of a complex data collection in a sequential manner without any need to know its underlying representation.*

But what really does that mean? Any ideas what complex data is and why its representation should be hidden from the client? Let’s look at an example, which is very close to the reality, how to design architecture and how iterators can be important (the code will be written in JavaScript, but, in fact, it doesn’t matter).

Incoming business information is the following:
>*There is a company ‘Vehicles tuning’. The tuning includes two processes: wheel replacement and painting. The service is available for a few vehicle groups like cars, motorbikes, trucks. The company has the ability to serve only one vehicle per process at one moment, also due to the tuning procedure, wheels should be changed before a vehicle is painted.*

![Image of scenario 1.1](https://cdn-images-1.medium.com/max/800/1*C40dvb-jS0qdgDCPqeyUDA.png)

1.1 The scenario is simple, take a car from the list, fix the wheels and paint.

Pretty enough, no? At first we need to re-phrase incoming statements due to the basic requirements of the app architecture like flexibility and scalability. The key points are:

* the tuning includes two processes, but ‘scalability’ of the company means having more at any moment. It can be improving an engine, installing spoilers, etc.
* vehicles are divided into groups, the number of groups can be changed.
* priorities of these groups can be established, i.e. first — cars, after — motorbikes, after — trucks.
* initial priorities of groups can be changed dynamically, and the priority of some exact vehicle can be increased dynamically within one group as well. E.g.: a car owner needs to get their vehicle as soon as possible.
* the order of processes is important (‘wheels’ before ‘painting’) but it can be changed.

The next step, we can try to define main program items. But, let’s do this in a bit unusual way. It’s a good technique to abstract out the code and imagine some simplified picture from the real life. OK, at first sight, obviously, program entities will be:

* Vehicle (and its sub-types: Car, Truck, Motorbikes)
* WheelsService [Fast John, worker]
* PaintingService [Creative Ann, painter]
* TuningCompany [Boss Mike, boss]

You probably noticed the names in the brackets, as I mentioned before, we can associate them with the program items, so, it will be easy to understand its usage and features. We can write a small plot here:

>*Fast John works with wheels replacement. He is fast, as he said at the interview, he worked in an F1 team some time ago. So, when he is changing wheels on the 3rd car, Creative Ann is still thinking what tone of grey she should pick up for ribbon borders for the 1st car. She is slower than John, she tried to be a painter in her childhood. Maybe because of this Ann doesn’t talk with John, to be honest, she even doesn’t know him. Boss Mike knows them both, but he is boss and he doesn’t care about the ribbons color, he likes spending time on the boat and playing with his dog Rex.*

![Image of scenario 1.2](https://cdn-images-1.medium.com/max/800/1*XW9hTv7bR1j7CXH3Xr5uew.png)

1.2 The main module is aware of existence of the two services, but not about their implementation.

>*“Think about it from another’s point of view, if I sell the company to Jim, will John and Ann be able to continue working? Or do I know too much about ‘ know-how’ ?”, -Mike can ask.*

The picture, of course, is still not completed. So, if we analyze the requirements more deeply, we can find out a few more items which will help us to collect the puzzle:

* VehiclePrioritiesListService [Responsible Sam, manager]
* Iterator (according to the article topic, we should have at least one)
* IteratorsFactory [Generous Tom, bro]
* ServiceDependenciesFactory [Smart Mark, engineer]

So.. looks better, but, who knows, we just think aloud, will see. Before we start to write some code lines, let’s wait a second and clarify new people roles:

>*Responsible Sam keeps the actual list of vehicles. New cars can be added, or one angry motorbike’s owner can ask for his bike immediately, so Sam will need to change priorities, and so on. In a few words — really responsible paper work. Generous Tom is the oldest man in the company. He is a ‘bro’: if you need something, you can feel free to ask him, he will give you everything you need. Everyone knows, likes and respects Tom. And Pedantic Mark: he’s graduated from the college recently. He is an engineer. He is aware of complex industrial processes and trying to apply them at his current work as well.*

![Image of scenario 1.3](https://cdn-images-1.medium.com/max/800/1*EQh1MxQoyR9TOQpvQwCSXA.png)

1.3 IteratorsFactory is the end-point for different services. It has access to the vehicle list. Complex dependencies are encapsulated inside a separate module.

Now, we can try to imagine the whole picture and describe the connections between different items of the application (people). But, let’s do this only for ‘painting service’ and omit ‘wheels service’ completely. For ‘wheels’ everything is almost the same and even easier. You will see that the services are loosely coupled and the app is open for extension.

>‘Loose coupling’, ‘open for extension’ are a few more important approaches which you should follow when designing your app. As Mike could say: ‘A module can be aware of ‘sand-box’ , but not about other modules. You can see this in my company, ‘wheels service’ has no references to ‘painting’ service, that’s cool’.

So, look at the picture below, some objects in the chain are omitted, just to the keep focus on key points.

![Image of scenario 1.4](https://cdn-images-1.medium.com/max/800/1*VTm8eIwYzqFqeCdAjcEJyg.png)

1.4 How Ann is working.

Let’s analyze the scheme above, as you can see it’s pretty straightforward:
* PaintingProcess gets the iterator of vehicles from IteratorsFactory.
* IteratorsFactory creates iterator based on the list received from PrioritiesListService
* PrioritiesListService generates a sorted list of vehicles. They can be retrieved from any sources and sorted in any order. Only  PrioritiesList really knows and cares about these data configurations.
* PaintingProcess starts processing only after the verification by DependeciesFactory.

As it was mentioned before, you can easily replace PaintingProcess with WheelsService .
By the way, you don’t see Boss Mike in the picture. Because the boss doesn’t work. He has invented company and that’s it.
> *The common mistake is to put a lot of code to the main controller.*

Let’s talk about this tuning process from customer’s (Vehicle) point of view. As you can understand from the scheme, a customer discusses the service with the Manager Sam, that’s it. Customer doesn’t go from the manager’s office to the employees or even the boss to discuss some details.

> *The next common mistake is data access end point. Don’t allow everyone to get data from the source. The layers are important. Only its direct parent should have the access to the data layer.*

Probably it’s becoming boring, let’s add a few lines of code. I will not write all code, just the lines which describe the key logic, as usual. Let’s start with the list of vehicles. It’s an end-point for customer, e.g. when customer Nick brings his car to the company, he is discussing this with manager Sam (as it has been already mentioned previously).

```javascript
var EventsMap = {
  NEW_VEHICLE_IS_ADDED: 'new-vehicle-is-added',
  LIST_IS_MODIFIED: 'list-is-modified',
  ...
};

var VehiclePrioritiesListService = {
  vehiclesListData: [{type: 'car', name: 'Honda Civic', owner: 'Barbara'}, ...],
  
  sortedListPrototype: [],
  
  getActualList: function() {
    return this.createCopy(this.sortedListPrototype);
  },
  
  registerVehicle: function(vehicle) {
    this.vehiclesListData.push(vehicle);
    
    this.rebuildSortedListPrototype();
    
    this.notifySubscribers(EventsMap.LIST_IS_MODIFIED);
  },
  
  rebuildSortedListPrototype: function() {
    this.sortedListPrototype = someComplexLogicDependingOnSorting(this.vehiclesListData);
  },
  
  on: function(eventName, cb) {
    this.eventListeners[eventName].push(callback);
  }
}
```
1.1 Basic implementation of PrioritiesList

Let’s talk about the code of the priorities list. The key points are:
* vehicles data is hard-coded for readability, but of course we can fetch it from any sources.
* the main method is getActualList, so, everyone who wants to use the sorted list of vehicles can just call it and receive a copy.
* when a new vehicle appears, we add it to the system by calling registerVehicle, then rebuild the list and notify subscribers. You can see it further, we will use the subscription API from outside.
* as you probably noticed, sorted list is named sortedListPrototype, and complex logic of sorting is applied once, and list instances are copies of it (there is no need to do complex sorting logic each time).
* I named the method as someComplexLogicDependingOnSorting, without implementation, that’s redundant here. As it was mentioned before, the sorting logic can be various, e.g. cars are first, or some exact vehicle of VIP client is first and so on.

The next file describes iterator’s logic. Let’s have a look at the code at first.

```javascript
class QueueIterator {
    constructor (config) {
        this.listSource = config.listSource;
        
        this.itemValidator = null;
        this.currentItem = null;
        
        this.rebuildQueue();
    }
    
    setItemValidator (itemValidator) {
        this.itemValidator = itemValidator;
    }
    
    getNext () {
        this.itemValidator.markItemAsCompleted(this.currentItem);

        if (this.queue.isDirty) {
            this.rebuildQueue();
        }

        this.currentItem = _.find(this.queue.prioritiesList, this.itemValidator.isItemReady);
        return this.currentItem;
    }
    
    rebuildQueue () {
        this.queue = {
            isDirty: false,
            prioritiesList: listSource.getActualList()
        }
    }
    
    markQueueAsDirty () {
        this.queue.isDirty = true
    }
}
```
1.2 Iterator’s main logic

We can analyze the listing above. The key points are:
* data source is set via class constructor
* item readiness validator can be set or changed dynamically. As you remember from the previous paragraphs, only engineer Mark is aware of when a vehicle is ready for the processing
* queue. It’s an interesting place here, we use not a plain list, but also flag which describes when list is ‘dirty’, i.e. it’s not actual and should be updated.

An instance of Queue iterator class will be provided to main processed by IteratorsFactory. Probably let’s see how it can be looking.

```javascript
//injections: QueueIterator, VehiclePrioritiesListService
var IteratorsFactory = {
    queueIteratorList: [],
    
    initListChangesHandling: function(events) {
        VehiclePrioritiesListService.on('list-is-modified', (event) => {
            this.resetIterators();
        });
    },
    
    resetIterators: function() {
        _.each(this.queueIteratorList, function(iterator) {
            iterator.markQueueAsDirty();
        });
    },
    
    createQueueIterator: function() {
        var queueIterator = new QueueIterator({
            listSource: VehiclePrioritiesListService
        });
        
        this.queueIteratorList.push(queueIterator);
        return queueIterator;
    }
}
```
1.3 Factory for iterators

As you could see, the previous class QueueIterator has no dependencies. If we take this as the main goal, we can easily avoid direct injections here in the factory as well, to be able to configure the iterator and the list, but I didn’t do this, to make code more understandable at first sight. If you agree, let’s move on to the key-point list for IteratorsFactory:
* createQueueIterator method which is used from Painting/Wheels services
* by calling initListChangesHandling we subscribe to changes in the list, like a new item is added/removed and reset iterators (make a queue ‘dirty’). As you can see we need to call this method from outside, or can skip it, if we don’t wish to handle changes dynamically.
* initListChangesHandling can be called with the parameters and specify which exact events we should monitor (let’s say, only when items are added). For readability this code is omitted as well and ‘list-is-modified’ is hard-coded (don’t be afraid).

Now we can go further and have a look at PaintingService:

```javascript
//injections: IteratorsFactory, ServiceDependenciesFactory
class PaintingService {
  constructor () {
    this.vehiclesIterator = IteratorsFactory.createQueueIterator();
    
    this.vehiclesIterator.setItemValidator(
      ServiceDependenciesFactory.getPaintingDependencyValidator()
    );
  }
  
  startWorking () {
    this.paintVehicles();
  }
  
  paintVehicles () {
    var vehicle = this.vehiclesIterator.getNext();
    
    if (!vehicle) {
      //finished with all vehicles.. 
    }
    
    doSomeLongProcessOfPainting().finally(()=> {
      this.paintVehicles();
    });
  }
}
```
1.4 Painting process, iterator usage

You see, we already use iterator! That’s cool. As usual, quick overview what we coded:
* get iterator from IteratorsFactory
* set validator which ‘‘knows’’ some business logic (‘painting after wheels’)
* paintingVehicles is recursive method, because we need to go through all vehicles
* actual work of service is inside doSomeLongProcessOfPainting, we don’t show it, it’s really doesn’t matter and can be implemented in any ways

What did we miss? It seems ServiceDependenciesFactory and the main controller. Let’s cover ServiceDependenciesFactory at first. As you remember from Iterator code (listing 1.2), we need two public methods for it.

```javascript
var ServiceDependenciesFactory = {
  
  getPaintingDependencyValidator: function() {
    return {
      markItemAsCompleted: function(item) {
        item.isPaintingCompleted = true;
      },
      
      isItemReady: function(item) {
        return !item.isPaintingCompleted && item.isWheelsCompleted;
      }
    };
  }
  
};
```
1.5 Dependencies service.

As you can see we simply described:
* how ‘set’ and ‘validate’ methods can look. Of course, they can be no so simple in another case, one more time, what we are trying to do, we are trying to make PaintingService code completely independent of WheelsService, and we have achieved that.
* validator for WheelsService (or any other) also can be described here, we will omit it.

And, finally, main controller:

```javascript
//injections: WheelsService, PaintingService
var services = [
  new WheelsService(),
  new PaintingService()
];

_.each(services, service=> service.startWorking());
```
1.6 Main controller is small

As you can see, Boss Mike really doesn’t care.

![Image of Boss](https://cdn-images-1.medium.com/max/800/1*4P0FyZJWlQ-7pSNDP_jWtQ.png)

1.5 Boss Mike doesn’t care

###In the end..
> *‘So, sounds funny, but how is all these stuff is related to iterators?’, — you can ask.*

Let’s see by summarizing main approaches:

1. Iterator data usually should be loosely coupled with the iterator itself. Check the priorities list, it contains much logic to keep the data state and iterator itself only does its direct work.

2. Iterator should iterate sequence and keep the current state. But it needs to be ‘‘smart’’ for data changes, check ‘dirty queue’ logic. Also, iterator functionality can be extended with feature like ‘wake up after still mode’ (when we have finished with all vehicles, but we want to start again if a new car appears). It’s easy to add, we can discuss if it’s needed.

3. Iterators can be requested from different places. You need to have a managing service which can control all of them. Please see ‘iterators factory’.

4. Third party business logic should be separated from iterator. The parallel business process can affect iterations flow, but iterator shouldn’t know why. Check dependencies service.

5. Iterator always work with sequence data, but never knows the real procedure how to use that data. It controls index in list, but not a moment when index needs to be changed. Check iterators usage in PaintingServices.

Also, you probably may say: no, the example is too far from real life of JS developer (I believe not only JS). Maybe. Let’s see how program architecture described above can be reusable in different business cases:
* data synchronization: imagine ‘drop-box’, a mail client and so on. You have folders that can be synchronized in some exact order (priorities list). Process of synchronization also can be complex and include a few threads. i.e. for drop-box (any files storage) 1st process is to sync files list per folder, 2nd process is to load content of files. Pretty the same as ‘wheels’ and ‘painting’, huh?
* complex operations/calculations with limited environment resources: probably you will try to find how you can do it in a sequence.
* any factory with conveyor. There is a list of operations which should be applied for some item in exact order before the items become ready for client usage (‘Vehicle tuning ’ is one of them in some sense).

*images are created with the icons from https://www.iconfinder.com/

