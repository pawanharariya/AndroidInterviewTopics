# Android Interview Topics
Recycler View Working
- There is a list of data, the adapter binds data into views and gives to LayoutManager, which controls the position on View on screen. 

- RecyclerView doesn’t allocate item view for each entry in data list, but only limited item views that fit the visible screen and reuses those item layout as user scrolls.

- Detaching - Items just above and below view port are detached views.

- Scrapping - Item layout scrolls out of screen and recycling starts and it becomes scrap view and put into scrap heap. When required they are directly passed to layout manager without passing the view back to adapter.

- Recycling - Recycle pool contains dirty view, views with incorrect data. To be reused they are passed again to adapter and then to layout manager. The adapter takes references of views from ViewHolder.

Recycler View Optimisations
1. DiffUtils

2. setHasFixedSize(true) : RecyclerView’s size doesn’t depend on size of its children(except number of items). RecyclerView can still change its size based on parent’s size. So RecyclerView’s size calculation doesn’t happen on each item change. For eg. TextView has fixed lines and use ellipsize.

3. setItemViewCacheSize(size) : number of offscreen views to retain before adding them to recycle pool. Default size is 2.

4. notifyItemChanged() and notifyItemRangeChanged() methods instead of full update

5. Using notifyItemChanged(payload : Object) - inside BindViewHolder check if payload is not empty and use payload to update instead of redrawing everything, if null make a full update.

6. ListLiveData implementation

7. ViewHolder Lifecycle - 
	- onCreateViewHolder
	- onBindViewHolder
	- onViewAttachedToWindow (ViewHolder becomes visible - play videos or audio)
	- onViewDetachedFromWindow (ViewHolder goes off screen - pause memory intensive events)
	- onViewRecycled (release resources held by ViewHolder)

8. Have Common RecycledViewPool for Nested RecyclerViews because Child RecyclerViews have same view layouts so they can be recycled from other child RecyclerViews. (Static RecyclerViewPool class)

9. Using StableIDs : setHasStableIds(true) on adapter and override getItemId(). If two items in the new and old dataset have the same ID (this is done by implementing the getItemId method), the same ViewHolder will be used for them and doesn’t update UI or use Item changed methods smartly after notifyDataSetChanged() call. Useful for animation purposes. 

Scoped Functions : 
Execute block of code with context of object. Provides scope to object with lambda expression, inside the scope we can reference the object without its name. let, run, with, also, apply

1. Differ based on how to refer the context object.
	- run, with, apply : use this as lambda receiver 
	- let, also : use it as lambda argument
2. Differ based on return value
	- run, with, let : return value of lambda (variable assignment or return some value from lambda)
	- apply, also : return caller object (use of chaining function calls)
Usage : 
1. let : non-null code block using safe operator object?.let{ it -> println(“$it is not null now)}
2. with : calling functions on object without return value (with this object do following)
3. run : object initialisation and returning some value 
4. apply : don’t return a value and operate on member variables of object (use for object initialisation)
5. also : operate on object and not its fields  eg. val something = Something().also{ println(it) }

ViewModel Working : 
We don’t instantiate view model in activity, it is provided by ViewModel provider. Every activity has a ViewModelStore that stores all ViewModels declared in the activity. This ViewModelStore is not destroyed on configuration change, hence ViewModel provider is able to provide the ViewModel. The ViewModel is destroyed when onCleared() is called, which doesn’t happen in case of configuration change. It is called in case Back Press or when activity’s finish() is called. 

Context in Android : 
It serves as bridge between our app and the Android system. Suppose there is Resources files, which is memory/file system of our app, so it is system component, now our wants to use resources files. So we need context to read this. Launch other apps, launch activities. Like adapter is a plain class, but if we want to write code to display Toast in the adapter itself, we need context object in adapter and then pass it to Toast. In this case the context is just the activity, so that Toast knows I need to display message it in this activity. So context is just a super class and the activity and application class are subclasses of it. ActivityContext and ApplicationContext have different lifetime same as corresponding activity and application. BroadCastReceiver and Content Providers are not context, but they get the context.

Memory Leaks : When the system stores some object in memory which is no longer required or can no longer be accessed is called memory leak.

1. Using activity context/view references in ViewModel. If current activity instance is destroyed due to configuration changes, the context/view references will be retained by ViewModel, because ViewModel is not destroyed on configuration change. Memory leak cannot happen because of ApplicationContext because it is singleton and there is only one object.

Why is there activity context when we have application context? 
There is activity context despite being application context because sometimes we need information about the activity itself.
1. Layout Inflation requires app theme, which application context doesn’t know.
2. Starting Activity as it requires back stack information, which application class doesn’t know.
3. Showing dialogs, because UI(activity) is still visible and we have to show dialog on top of Activity only.

2. Static reference to the activity inside the activity itself : During config changes, Android will try to destroy(garbage collect) the activity but it cannot because there is reference to it. Anyway new instance of Activity is created for the app, but old instances still remain in memory.

Steps to Avoid : 
1. Avoid using context beyond its lifecycle. 
2. Destroying resources like FusedLocationListener, Timers, in onDestroy(). 
3. Avoid usage of static references of inner class, because it has reference to outer class.
4. Using application context for singletons and initialisation in library.

Tasks, Back Stack and Launch Modes 
Back Stack - Stack of screens or activities.
Task - It is back stack and the activities themselves.
Launch Modes - Define the behaviour when new Activity is added to back stack.

	1. Standard (default) - new instance of activity is pushed on back stack. Browser has some activities and we open a link that opens browser, so new activity is pushed on top of previous activities. A->B->C->D->B possible if B activity is launched from same app or different app.

	2. Single Top - Go to same browser activity instance and no new activity is created (if it is at top of back stack) prevents back button fatigue. Starting same activity from current activity like search results page and search again page, so each activity is not piled up.

	3. Single Task - Each new instance is launched in previous task if exits. For example, we have an app with multiple activities open, now we switched to different app that somehow navigates to previous app. In this case, the activity is launched on same task and clears top activities(if any) to get to required activity. Pressing back will go to previous activity on current app, and not the app we were using. (OnNewIntent() callback). For separate task, we also need task affinity.

	4. Single Instance - launches new instance of activity in new task, and no other activities will be created in current task. For example Payment app will show only payment link and not other parts of app. If we launching a single instance in same app, it will still go to different task and other non-single instance activities will continue in current task.

TaskStackBuilder - To create a fake back stack. Opening a notification into deep app, so when back presses it should behave as if, we are using the app, and creating fake activities such as we feel as if we navigated there, instead of back press taking as to launcher Home Screen.


Kotlin Specifics : 

1. init Block : after primary constructor before secondary constructor and does not take parameters. Can have multiple init block and they run in order in which they are defined. 

2. const :  compile time constants, reduces object over head at runtime because at compilation, variable is replaced with the exact value.

3. Inline function : replaces the function call with function body at compilation, less function call overhead at runtime. Best for small code that is used limited number of times. (But is written separately for clarity).

## Activity Lifecycle ##

1. Single Activity Lifecycle

Case 1 : App finish and restarted - using Back button or onFinish() call

- Normal activity startup : onCreate -> onStart -> onResume
- Back button press : onPause -> onStop -> onDestroy  
- Activity Restarted : onCreate(null) -> onStart -> onResume
  
-- onSaveInstanceState is not called because activity is finished

Case 2 : User navigates away - user press Home Button

- Normal activity startup : onCreate -> onStart -> onResume
- Home Pressed : onPause -> onStop -> onSaveInstanceState
- App restarted : onRestart -> onStart -> onResume

Case 3 : Configuration Changes

- Normal activity startup : onCreate -> onStart -> onResume
- Device Rotated : onPause -> onStop -> onSaveInstanceState -> onDestroy -> onCreate(Bundle) -> onStart -> onRestoreInstanceState -> onResume
  
-- State needs to be saved and restored so that users can continue exactly where they left off

Case 4 : App is paused by the system - By runtime permission dialog, intent chooser share dialog, third party login dialog, multi-window mode (App loses focus but partially visible) 

- Normal activity startup : onCreate -> onStart -> onResume
- App got interfered (focus lost) : onPause
- Focus comes back to app : onResume
  
-- Doesn’t apply to dialogs in same app (Alert Dialog) or notification tray expanded by user


NOTE : onRestoreInstanceState - Mostly onCreate(bundle) is used, but it is sometimes convenient to do it here after all of the initialisation has been done.

2. Multiple Activities Lifecycle

Case 1: Navigation between activities

- Activity 1 Startup : onCreate -> onStart -> onResume
- Activity Switch : 
	Activity 1 : onPause -> onStop -> onSaveInstanceState (parallel)
	Activity 2 : onCreate -> onStart -> onResume (parallel)
- Back press from Activity 2 : 
	Activity 1 : onRestart -> onStart -> onResume (parallel)
	Activity 2 : onPause -> onStop -> onDestroy (parallel)
- Back press from Activity 1 : onPause -> onStop -> onDestroy
  
-- State is saved for activity 1, so that if configuration change happens in activity 2 or if system kills the app, state can be restored

Case 2: Configuration change with activities on back stack
- Activity 2 opened : 
	  Activity 1 : onPause -> onStop -> onSaveInstanceState (parallel) 
	  Activity 2 : onCreate -> onStart -> onResume (parallel)
- Configuration Change :
	Activity 2 : onPause -> onStop -> onSaveInstanceState -> onDestroy -> 	onCreate(Bundle) -> onStart -> onRestoreInstanceState -> onResume
- Back Pressed from Activity 2 : 
	  Activity 1 : onDestroy -> onCreate(Bundle) -> onStart -> 	onRestoreInstanceState -> onResume (parallel)
	  Activity 2 : onPause -> onStop -> onDestroy (parallel)

-- Saving state is not only important for the activity in the foreground. All activities in the stack need to restore state after a configuration change to recreate their UI.

Case 3: App’s process killed with activities on back stack
- Activity 2 opened : 
	Activity 1 : onPause -> onStop -> onSaveInstanceState (parallel) 
	Activity 2 : onCreate -> onStart -> onResume (parallel)
- Home pressed : 
	Activity 2 : onPause -> onStop -> onSaveInstanceState
- System kills process and App reopens
	Activity 2 : onCreate(Bundle) -> onStart -> onRestoreInstanceState -> onResume
- Back pressed : 
	Activity 1 : onCreate(Bundle) -> onStart -> onRestoreInstanceState -> onResume (parallel)
	Activity 2 : onPause -> onStop -> onDestroy (parallel)

-- State of the full back stack is saved but, in order to efficiently use resources, activities are only restored when they are recreated.

Coroutines
1. Light weight thread and execute on top of threads (Because normal thread are limited and expensive) but with coroutines threads can be freed if waiting.
2. Multiple coroutines can be executed on single thread, because if the task is taking long time, thread can be freed.

Coroutine scope and Coroutine context and Dispatchers.
Scope - Defines lifetime of coroutine. So that all coroutines associated with the scope can be cancelled.
Context - Defines threads on which coroutine will run with the help of dispatchers.

Dispatchers : Dispatch the coroutine on Threads. Dispatchers.Main, Dispatchers.IO

Suspending Functions : suspends the execution on current thread and frees the thread. Suspending function should be called from coroutine or other suspending function. For making callback based code to sequential code.

Job - It helps to manage coroutine and cancel it or wait for it. Job object is returned when we create the coroutine. And we can use job.join() to wait for its completion.

Join : Job.join() wait till coroutine is finished.

Launch : Fire and forget coroutine, we don’t care about result (returns JOB instance)

Async/Await : coroutine executed using Async returns a value and we can wait for this value till it’s returned. (Returns Deferred<T> subclass of Job)

Inside a coroutine suspending functions run synchronously so we can wait for result. But outside we have to use join or async-await. If there are multiple suspending functions and we want to run them in parallel inside a coroutine either we can execute them in separate coroutines altogether or execute them separately inside the main coroutine using launch and async-await.

Jobs have parent-child relationship. If we execute coroutines inside a coroutine they become child jobs. Child jobs inherit the context of parent job and are cancelled if parent job is cancelled. But they can have their own context as well by explicitly defining it. We can check if the coroutine is still Active and then cancel based on it.

Parent job cancel the child coroutine if it returns CancellationException if it return some other exception entire coroutine is cancelled.

withContext() : blocks the coroutine till it gets completed and then executes the later commands

runBlocking : it block the thread until coroutine is completed, the other commands will be be executed till the coroutine is completed but the program won’t exit till coroutine is complete. Useful in test cases.

ViewModelScope is attached to ViewModel. Lifecycle Scope is attached to activities or fragments.

Dependency Injection
Without DI :
1. Code is not unit testable.
2. Non extensible code because dependency cannot be changed later.
3. Violate single responsibility principle, creating other objects.
4. Lifetime problem (Database requires single connections but every class creates its own connection) (Non-reusable). 

Constructor Injection and Field Injection

Dagger 2 : Helps to implement DI
1. Consumer - @Inject
2. Producer - @Module, @Provides, @Binds
3. Connector - @Component (connects producer and consumer)

Dagger-Hilt
We get components and scopes by default. Dependencies defined in parent component are available to child component by default.
Scopes define boundary. Like component will be singleton in activity with @ActivityScopre
Runtime dependencies are available	automatically like ApplicationContext.

Constructor Injection cannot be used in case of Third party libraries, on interfaces or Builder patter object creation. So we use modules and define how to create objects of it.

@InstallIn attaches the module to component. Hilt provides Application and Activity component by default. Application component is Singleton component. Components follow hierarchy.

@Binds to bind an interface with its implementation.

Qualifiers help to resolve binding in case of multiple implementations of same type, then which object to return. @Named(“name”) is used.

@HiltViewModel automatically creates ViewModel Factory and injects dependencies.

ViewModel Working : 
We don’t instantiate view model in activity, it is provided by ViewModel provider. Every activity has a ViewModelStore that stores all ViewModels declared in the activity. This ViewModelStore is not destroyed on configuration change, hence ViewModel provider is able to provide the ViewModel. The ViewModel is destroyed when onCleared() is called, which doesn’t happen in case of configuration change. It is called in case Back Press or when activity’s finish() is called. 

Context in Android : 
It serves as bridge between our app and the Android system. Suppose there is Resources files, which is memory/file system of our app, so it is system component, now our wants to use resources files. So we need context to read this. Launch other apps, launch activities. Like adapter is a plain class, but if we want to write code to display Toast in the adapter itself, we need context object in adapter and then pass it to Toast. In this case the context is just the activity, so that Toast knows I need to display message it in this activity. So context is just a super class and the activity and application class are subclasses of it. ActivityContext and ApplicationContext have different lifetime same as corresponding activity and application. BroadCastReceiver and Content Providers are not context, but they get the context.

Memory Leaks : When the system stores some object in memory which is no longer required or can no longer be accessed is called memory leak.

1. Using activity context/view references in ViewModel. If current activity instance is destroyed due to configuration changes, the context/view references will be retained by ViewModel, because ViewModel is not destroyed on configuration change. Memory leak cannot happen because of ApplicationContext because it is singleton and there is only one object.

Why is there activity context when we have application context? 
There is activity context despite being application context because sometimes we need information about the activity itself.
1. Layout Inflation requires app theme, which application context doesn’t know.
2. Starting Activity as it requires back stack information, which application class doesn’t know.
3. Showing dialogs, because UI(activity) is still visible and we have to show dialog on top of Activity only.

2. Static reference to the activity inside the activity itself : During config changes, Android will try to destroy(garbage collect) the activity but it cannot because there is reference to it. Anyway new instance of Activity is created for the app, but old instances still remain in memory.

Steps to Avoid : 
1. Avoid using context beyond its lifecycle. 
2. Destroying resources like FusedLocationListener, Timers, in onDestroy(). 
3. Avoid usage of static references of inner class, because it has reference to outer class.
4. Using application context for singletons and initialisation in library.

Tasks, Back Stack and Launch Modes 
Back Stack - Stack of screens or activities.
Task - It is back stack and the activities themselves.
Launch Modes - Define the behaviour when new Activity is added to back stack.

	1. Standard (default) - new instance of activity is pushed on back stack. Browser has some activities and we open a link that opens browser, so new activity is pushed on top of previous activities. A->B->C->D->B possible if B activity is launched from same app or different app.

	2. Single Top - Go to same browser activity instance and no new activity is created (if it is at top of back stack) prevents back button fatigue. Starting same activity from current activity like search results page and search again page, so each activity is not piled up.

	3. Single Task - Each new instance is launched in previous task if exits. For example, we have an app with multiple activities open, now we switched to different app that somehow navigates to previous app. In this case, the activity is launched on same task and clears top activities(if any) to get to required activity. Pressing back will go to previous activity on current app, and not the app we were using. (OnNewIntent() callback). For separate task, we also need task affinity.

	4. Single Instance - launches new instance of activity in new task, and no other activities will be created in current task. For example Payment app will show only payment link and not other parts of app. If we launching a single instance in same app, it will still go to different task and other non-single instance activities launched later will continue in current task.

TaskStackBuilder - To create a fake back stack. Opening a notification into deep app, so when back presses it should behave as if, we are using the app, and creating fake activities such as we feel as if we navigated there, instead of back press taking as to launcher Home Screen.


Kotlin Specifics : 

1. init Block : after primary constructor before secondary constructor and does not take parameters. Can have multiple init block and they run in order in which they are defined. 

2. const :  compile time constants, reduces object over head at runtime because at compilation, variable is replaced with the exact value.

3. Inline function : replaces the function call with function body at compilation, less function call overhead at runtime. Best for small code that is used limited number of times. (But is written separately for clarity).

4. JvmStatic Annotation : function inside object class in Kotlin. Calling function in Kotlin class.fun() but not possible in java, Java required class.INSTANCE.fun(). So @JvmStatic creates additional static function so that the function can be called from JAVA. Now we can call class.fun() in java also. (Interoperability)

5. JvmField Annotation : Exposes the field instead of generating getter and setter. dataClass.field is possible in Kotlin but not Java. In Java we have to use dataClass.getField() which is auto generated by Kotlin, but with @JvmField Annotation we can use dataClass.field in Java also.

6. JvmOverloads Annotation : Kotlin has default parameters and Java has method overloads. So we cannot call Koltin method with default parameters directly in Java. Using @JvmOverloads compiler generates overloading functions that substitutes the default parameters. Now we can call the Kotlin function that has default parameters. 

7. == structural equality(requires onEquals() method overridden) === referential equality.

8. Companion Object : Declare static properties and methods and constants inside class, each class can have only one companion object. Companion object can have init blocks. They are not inherited by sub classes unlike static methods in Java


