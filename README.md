# call-list-to-queue-voximplant
How to build a call queue

Our platform can accept multiple incoming calls, but a human operator can speak only with one person at a time. This can be solved by a call queue: while the operator speaks with a user, all other users are waiting in a queue, with optional waiting music being played and/or the synthesized message informing the users about the remaining waiting time.

Voximplant provides the Call Queue abstraction to automatically handle a queue of incoming calls.
Creating a queue

    Open Applications in the Voximplant control panel.
    Go to your application and switch to the Queues tab in it. The queue control interface will be displayed.
    Click Create queue in the upper right corner (or Create in the center of the screen). At this step, name the queue ‘main’, assign at least one user, and click Add to confirm the operation. Note that a queue name should be unique within your Voximplant developer account.

Code

Switch to the Scenarios tab and create a "queue" scenario with the following code:
forwarding calls to a queue

First, we mount the ACD module, then create a handler for an incoming call. Inside a handler we answer a call and when it's connected, put this call to the queue via the enqueueACDRequest method. When an operator is reached, i.e., answers a call, we connect the caller with this operator. Finally, we place the handlers for cases when an operator rejects a call and either side of conversation hangs up.
Waiting in a queue

By default, a user will hear nothing while waiting in the queue, since we have not instructed VoxEngine to produce any audio. Good practice is to inform the user about their queue position and play some music to indicate the call in progress.

Let’s add the background music by using the call object’s startPlayback method. In order to start background music, update the previous code as follows (lines 11 and 12):
