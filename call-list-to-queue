require(Modules.CallList);
require(Modules.AI);
require(Modules.ACD);

const callerId = "551131972019";
let playbackCounter = 0;
let customerData = undefined;

VoxEngine.addEventListener(AppEvents.Started, function(e){

    let data = VoxEngine.customData();
    
    Logger.write(`data received: ${data}`);

    customerData = JSON.parse(data);

    Logger.write(`data parsed: ${JSON.stringify(customerData)}`);

    Logger.write(`Calling ${customerData.first_name} ${customerData.last_name} on ${customerData.phone}`);

    let info = PhoneNumber.getInfo(customerData.phone);
    Logger.write(`some to call info number: ${info.number}, type: ${info.numberType}, region: ${info.region}`);


    // calling
    var outCall = VoxEngine.callPSTN(customerData.phone, callerId);

    // Trying to detect voicemail
    outCall.addEventListener(CallEvents.AudioStarted, function(){
        AI.detectVoicemail(outCall)
        .then(voicemailDetected)
        .catch(()=>{
            Logger.write('Voicemail not detected');
        })
    });

    // Add event listeners
  outCall.addEventListener(CallEvents.Connected, handleCallConnected);
  outCall.addEventListener(CallEvents.Failed, handleCallFailed);
  outCall.addEventListener(CallEvents.Disconnected, handleCallDisconnected);
});

function handleCallConnected (e) {
    Logger.write(`Connected call with ${customerData.first_name} ${customerData.last_name} on ${customerData.phone} `);

    // setTimeout(() => {

    //     e.call.say(`Hola ${customerData.first_name}, nuestros asesores te antenderan en breve`, Language.MX_SPANISH_FEMALE);
    // }, 500);

    // e.call.addEventListener("Call.PlaybackFinished", handlePlaybackFinished);

    //Incoming is a name of the queue that was created in Control Panel
    var request = VoxEngine.enqueueACDRequest("test-out-auto-call-list", customerData.phone, {
        headers: { "X-header": "title"},
        priority: 1
    });
	
    request.addEventListener(ACDEvents.Queued, function (a) {
      request.getStatus();
    });
    
    request.addEventListener(ACDEvents.Offline, function (a) {
    	e.call.say("Lo sentimos, en este momento no podemos procesar tu llamada. Adiós", Language.MX_SPANISH_FEMALE); 
      	e.call.addEventListener(CallEvents.PlaybackFinished, function(b) {
          VoxEngine.terminate();
      	});
    });
    
    request.addEventListener(ACDEvents.Waiting, function (a) {
      
      e.call.say(`Hola ${customerData.first_name}, en este momento estas en cola de espera en la posición ${a.position}. 
      Serás atendido por uno de nuestros operadores en ${(a.ewt+1)}  minutos.`, Language.MX_SPANISH_FEMALE);
    });

    e.call.addEventListener(CallEvents.PlaybackFinished, function (e) {
    	e.call.startPlayback("http://cdn.voximplant.com/toto.mp3", true);  
    });

    
    request.addEventListener(ACDEvents.OperatorReached, function (a) {
        
        //Operator is ready, let's send audio between calls
        VoxEngine.sendMediaBetween(a.operatorCall, e.call);
            a.operatorCall.addEventListener(CallEvents.Disconnected, VoxEngine.terminate);
            clearInterval(statusInterval);
    });
    

    e.call.addEventListener(CallEvents.OnHold, function(a) {
        e.call.startPlayback("http://cdn.voximplant.com/toto.mp3", true);  
    });

    e.call.addEventListener(CallEvents.OffHold, function(a) {
        e.callstopPlayback();  
    });
    
    
    var statusInterval = setInterval(function (){
      request.getStatus();
    }, 30000);
}

function handlePlaybackFinished(e) {
    e.call.removeEventListener(CallEvents.PlaybackFinished, handlePlaybackFinished);
    playbackCounter++;

    Logger.write(`handlePlaybackFinished playbackCounter: ${playbackCounter}`);

    // If the message was played 3 times - hangup
    if (playbackCounter === 3) {
        Logger.write(`playbackCounter reached hanging up`);
        e.call.hangup();
    } else {
        // Play one more time
        setTimeout(function () {
            Logger.write(`running playback once again :(`);
            e.call.say(`Hola ${customerData.first_name}, en este momento estas en cola de espera, nuestros asesores te antenderan en breve`, 
                Language.MX_SPANISH_FEMALE);
            e.call.addEventListener(CallEvents.PlaybackFinished, handlePlaybackFinished);
        }, 2000);
    }
}

function handleCallDisconnected(e) {
  // Tell CallList processor about successful call result
  CallList.reportResult({
    result: true,
    duration: e.duration,
    rating: 5,
  }, VoxEngine.terminate);
}

function handleCallFailed(e) {
  // Tell CallList processor that we couldn't get call connected
  // depending on the request options it will either try to launch the scenario again after some time
  // or will write the result (failed call) into result_data column of the CSV file with results
  CallList.reportError({
    result: false,
    msg: 'Failed',
    code: e.code
  }, VoxEngine.terminate);
}

function voicemailDetected(e) {
  // Voicemail?
  if (e.confidence >= 75) {
    VoxEngine.CallList.reportError('Voicemail', VoxEngine.terminate);
  }
}
