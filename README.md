# Activity-life-cycle-In-Android-studio

As a user navigates through, out of, and back to your app, the Activity instances in your app transition through different states in their lifecycle. The Activity class provides a number of callbacks that let the activity know when a state changes or that the system is creating, stopping, or resuming an activity or destroying the process the activity resides in.

Within the lifecycle callback methods, you can declare how your activity behaves when the user leaves and re-enters the activity. For example, if you're building a streaming video player, you might pause the video and terminate the network connection when the user switches to another app. When the user returns, you can reconnect to the network and let the user resume the video from the same spot.

Each callback lets you perform specific work that's appropriate to a given change of state. Doing the right work at the right time and handling transitions properly make your app more robust and performant. For example, good implementation of the lifecycle callbacks can help your app avoid the following:

Crashing if the user receives a phone call or switches to another app while using your app.
Consuming valuable system resources when the user is not actively using it.
Losing the user's progress if they leave your app and return to it at a later time.
Crashing or losing the user's progress when the screen rotates between landscape and portrait orientation.





##Activity-lifecycle concepts
To navigate transitions between stages of the activity lifecycle, the Activity class provides a core set of six callbacks: onCreate(), onStart(), onResume(), onPause(), onStop(), and onDestroy(). The system invokes each of these callbacks as the activity enters a new state.

 In some cases, the activity is only partially dismantled and still resides in memory, such as when the user switches to another app. In these cases, the activity can still come back to the foreground.

If the user returns to the activity, it resumes from where the user left off. With a few exceptions, apps are restricted from starting activities when running in the background.




### 1)  onCreate()
You must implement this callback, which fires when the system first creates the activity. On activity creation, the activity enters the Created state. In the onCreate() method, perform basic application startup logic that happens only once for the entire life of the activity.

For example, your implementation of onCreate() might bind data to lists, associate the activity with a ViewModel, and instantiate some class-scope variables. This method receives the parameter savedInstanceState, which is a Bundle object containing the activity's previously saved state. If the activity has never existed before, the value of the Bundle object is null.

If you have a lifecycle-aware component that is hooked up to the lifecycle of your activity, it receives the ON_CREATE event. The method annotated with @OnLifecycleEvent is called so your lifecycle-aware component can perform any setup code it needs for the created state.

As an alternative to defining the XML file and passing it to setContentView(), you can create new View objects in your activity code and build a view hierarchy by inserting new View objects into a ViewGroup. You then use that layout by passing the root ViewGroup to setContentView(). For more information about creating a user interface, see the user interface documentation.

Your activity does not remain in the Created state. After the onCreate() method finishes execution, the activity enters the Started state and the system calls the onStart() and onResume() methods in quick succession.

###  onStart()
When the activity enters the Started state, the system invokes onStart(). This call makes the activity visible to the user as the app prepares for the activity to enter the foreground and become interactive. For example, this method is where the code that maintains the UI is initialized.

When the activity moves to the Started state, any lifecycle-aware component tied to the activity's lifecycle receives the ON_START event.

The onStart() method completes quickly and, as with the Created state, the activity does not remain in the Started state. Once this callback finishes, the activity enters the Resumed state and the system invokes the onResume() method.

### onResume()
When the activity enters the Resumed state, it comes to the foreground, and the system invokes the onResume() callback. This is the state in which the app interacts with the user. The app stays in this state until something happens to take focus away from the app, such as the device receiving a phone call, the user navigating to another activity, or the device screen turning off.

When the activity moves to the Resumed state, any lifecycle-aware component tied to the activity's lifecycle receives the ON_RESUME event. This is where the lifecycle components can enable any functionality that needs to run while the component is visible and in the foreground, such as starting a camera preview.

When an interruptive event occurs, the activity enters the Paused state and the system invokes the onPause() callback.

If the activity returns to the Resumed state from the Paused state, the system once again calls the onResume() method. For this reason, implement onResume() to initialize components that you release during onPause() and to perform any other initializations that must occur each time the activity enters the Resumed state.


Here is an example of a lifecycle-aware component that accesses the camera when the component receives the ON_RESUME event:


Java

public class CameraComponent implements LifecycleObserver {

    ...

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void initializeCamera() {
        if (camera == null) {
            getCamera();
        }
    }
    ...
}
The preceding code initializes the camera once the LifecycleObserver receives the ON_RESUME event. In multi-window mode, however, your activity might be fully visible even when it is in the Paused state. For example, when the app is in multi-window mode and the user taps the window that does not contain your activity, your activity moves to the Paused state.

If you want the camera active only when the app is Resumed (visible and active in the foreground), then initialize the camera after the ON_RESUME event demonstrated previously. If you want to keep the camera active while the activity is Paused but visible, such as in multi-window mode, then initialize the camera after the ON_START event.

However, having the camera active while your activity is Paused might deny access to the camera to another Resumed app in multi-window mode. Sometimes it is necessary to keep the camera active while your activity is Paused, but it might actually degrade the overall user experience if you do.

For this reason, think carefully about where in the lifecycle it is most appropriate to take control of shared system resources in the context of multi-window mode. To learn more about supporting multi-window mode, see Multi-window support.

Regardless of which build-up event you choose to perform an initialization operation in, make sure to use the corresponding lifecycle event to release the resource. If you initialize something after the ON_START event, release or terminate it after the ON_STOP event. If you initialize after the ON_RESUME event, release after the ON_PAUSE event.

The preceding code snippet places camera initialization code in a lifecycle-aware component. You can instead put this code directly into the activity lifecycle callbacks, such as onStart() and onStop(), but we don't recommend this. Adding this logic to an independent, lifecycle-aware component lets you reuse the component across multiple activities without having to duplicate code. To learn how to create a lifecycle-aware component, see Handling Lifecycles with Lifecycle-Aware Components.

### onPause()
The system calls this method as the first indication that the user is leaving your activity, though it does not always mean the activity is being destroyed. It indicates that the activity is no longer in the foreground, but it is still visible if the user is in multi-window mode. There are several reasons why an activity might enter this state:

An event that interrupts app execution, as described in the section about the onResume() callback, pauses the current activity. This is the most common case.
In multi-window mode, only one app has focus at any time, and the system pauses all the other apps.
The opening of a new, semi-transparent activity, such as a dialog, pauses the activity it covers. As long as the activity is partially visible but not in focus, it remains paused.
When an activity moves to the Paused state, any lifecycle-aware component tied to the activity's lifecycle receives the ON_PAUSE event. This is where the lifecycle components can stop any functionality that does not need to run while the component is not in the foreground, such as stopping a camera preview.

Use the onPause() method to pause or adjust operations that can't continue, or might continue in moderation, while the Activity is in the Paused state, and that you expect to resume shortly.

You can also use the onPause() method to release system resources, handles to sensors (like GPS), or any resources that affect battery life while your activity is Paused and the user does not need them.

However, as mentioned in the section about onResume(), a Paused activity might still be fully visible if the app is in multi-window mode. Consider using onStop() instead of onPause() to fully release or adjust UI-related resources and operations to better support multi-window mode.

The following example of a LifecycleObserver reacting to the ON_PAUSE event is the counterpart to the preceding ON_RESUME event example, releasing the camera that initializes after the ON_RESUME event is received:


Java

public class JavaCameraComponent implements LifecycleObserver {
    ...
    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void releaseCamera() {
        if (camera != null) {
            camera.release();
            camera = null;
        }
    }
    ...
}
This example places the camera release code after the ON_PAUSE event is received by the LifecycleObserver.

onPause() execution is very brief and does not necessarily offer enough time to perform save operations. For this reason, don't use onPause() to save application or user data, make network calls, or execute database transactions. Such work might not complete before the method completes.

Instead, perform heavy-load shutdown operations during onStop(). For more information about suitable operations to perform during onStop(), see the next section. For more information about saving data, see the section about saving and restoring state.

Completion of the onPause() method does not mean that the activity leaves the Paused state. Rather, the activity remains in this state until either the activity resumes or it becomes completely invisible to the user. If the activity resumes, the system once again invokes the onResume() callback.

If the activity returns from the Paused state to the Resumed state, the system keeps the Activity instance resident in memory, recalling that instance when the system invokes onResume(). In this scenario, you don’t need to re-initialize components created during any of the callback methods leading up to the Resumed state. If the activity becomes completely invisible, the system calls onStop().

### onStop()
When your activity is no longer visible to the user, it enters the Stopped state, and the system invokes the onStop() callback. This can occur when a newly launched activity covers the entire screen. The system also calls onStop() when the activity finishes running and is about to be terminated.

When the activity moves to the Stopped state, any lifecycle-aware component tied to the activity's lifecycle receives the ON_STOP event. This is where the lifecycle components can stop any functionality that does not need to run while the component is not visible on the screen.

In the onStop() method, release or adjust resources that are not needed while the app is not visible to the user. For example, your app might pause animations or switch from fine-grained to coarse-grained location updates. Using onStop() instead of onPause() means that UI-related work continues, even when the user is viewing your activity in multi-window mode.

Also, use onStop() to perform relatively CPU-intensive shutdown operations. For example, if you can't find a better time to save information to a database, you might do so during onStop(). The following example shows an implementation of onStop() that saves the contents of a draft note to persistent storage:

Java

@Override
protected void onStop() {
    // Call the superclass method first.
    super.onStop();

    // Save the note's current draft, because the activity is stopping
    // and we want to be sure the current note progress isn't lost.
    ContentValues values = new ContentValues();
    values.put(NotePad.Notes.COLUMN_NAME_NOTE, getCurrentNoteText());
    values.put(NotePad.Notes.COLUMN_NAME_TITLE, getCurrentNoteTitle());

    // Do this update in background on an AsyncQueryHandler or equivalent.
    asyncQueryHandler.startUpdate (
            mToken,  // int token to correlate calls
            null,    // cookie, not used here
            uri,    // The URI for the note to update.
            values,  // The map of column names and new values to apply to them.
            null,    // No SELECT criteria are used.
            null     // No WHERE columns are used.
    );
}
The preceding code sample uses SQLite directly. However, we recommend using Room, a persistence library that provides an abstraction layer over SQLite. To learn more about the benefits of using Room and how to implement Room in your app, see the Room Persistence Library guide.

When your activity enters the Stopped state, the Activity object is kept resident in memory: it maintains all state and member information, but is not attached to the window manager. When the activity resumes, it recalls this information.

You don’t need to re-initialize components created during any of the callback methods leading up to the Resumed state. The system also keeps track of the current state for each View object in the layout, so if the user enters text into an EditText widget, that content is retained so you don't need to save and restore it.

Note: Once your activity is stopped, the system might destroy the process that contains the activity if the system needs to recover memory. Even if the system destroys the process while the activity is stopped, the system still retains the state of the View objects, such as text in an EditText widget, in a Bundle—a blob of key-value pairs—and restores them if the user navigates back to the activity. For more information about restoring an activity to which a user returns, see the section about saving and restoring state.

From the Stopped state, the activity either comes back to interact with the user, or the activity is finished running and goes away. If the activity comes back, the system invokes onRestart(). If the Activity is finished running, the system calls onDestroy().

### onDestroy()
onDestroy() is called before the activity is destroyed. The system invokes this callback for one of two reasons:

The activity is finishing, due to the user completely dismissing the activity or due to finish() being called on the activity.
The system is temporarily destroying the activity due to a configuration change, such as device rotation or entering multi-window mode.
When the activity moves to the destroyed state, any lifecycle-aware component tied to the activity's lifecycle receives the ON_DESTROY event. This is where the lifecycle components can clean up anything they need to before the Activity is destroyed.

Instead of putting logic in your Activity to determine why it is being destroyed, use a ViewModel object to contain the relevant view data for your Activity. If the Activity is recreated due to a configuration change, the ViewModel does not have to do anything, since it is preserved and given to the next Activity instance.

If the Activity isn't recreated, then the ViewModel has the onCleared() method called, where it can clean up any data it needs to before being destroyed. You can distinguish between these two scenarios with the isFinishing() method.

If the activity is finishing, onDestroy() is the final lifecycle callback the activity receives. If onDestroy() is called as the result of a configuration change, the system immediately creates a new activity instance and then calls onCreate() on that new instance in the new configuration.

The onDestroy() callback releases all resources not released by earlier callbacks, such as onStop().

Activity state and ejection from memory
The system kills processes when it needs to free up RAM. The likelihood of the system killing a given process depends on the state of the process at the time. Process state, in turn, depends on the state of the activity running in the process. Table 1 shows the correlations among process state, activity state, and the likelihood of the system killing the process. This table only applies if a process is not running other types of application components.

Likelihood of being killed	Process state	Final activity state
Lowest	Foreground (having or about to get focus)	Resumed
Low	Visible (no focus)	Started/Paused
Higher	Background (invisible)	Stopped
Highest	Empty	Destroyed
Table 1. Relationship between process lifecycle and activity state.

The system never kills an activity directly to free up memory. Instead, it kills the process the activity runs in, destroying not only the activity but everything else running in the process as well. To learn how to preserve and restore your activity's UI state when system-initiated process death occurs, see the section about saving and restoring state.

The user can also kill a process by using the Application Manager, under Settings, to kill the corresponding app.

For more information about processes, see Processes and threads overview.
