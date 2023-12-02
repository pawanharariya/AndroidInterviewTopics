# Android Interview Topics
## Recycler View Optimisation Techniques ##
1. DiffUtils
2. setHasFixedSize
3. notifyItemChanged() methods
4. Using notifyItemChanged(payload : List<Any>()) - inside BindViewHolder check if payload is empty else use payload to update instead of redrawing everything.
5. ListLiveData implementation - [Read more here](https://medium.com/google-developer-experts/notifying-recyclerview-on-a-specific-change-b36e6dc59e0f)

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
