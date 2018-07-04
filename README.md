# Libraires-gestion-Events
## Librairies développées pour la gestion des Events (+ plugins)

### Brique principale
````js

var manageEvents = {
    myevents : new Array(),
    $dc: function (id){
        var elem = null;
        if (document.getElementById(id) !== null) elem = document.getElementById(id);
        if(elem == null) console.log('%cERREUR : id "' + id + '" introuvable','color:#ff1d00;font-weight:bold');
        return elem;
    },
    listenClass:function(cls, evt, callback,capture){
        var elem = null;
        var sel = '.'+cls;
        var nodes = document.querySelectorAll(sel);
        if (nodes.length == 0) return null;
        for(var i = 0; i < nodes.length; i++){
            this.listenerAdd(nodes[i], evt, callback, capture);
        }
    },
    // c et t = current et total images chargées pour manageEventsImageLoader
	// e = event
	// trgt = target where the eventis applied
	// target = current target (where click is fired)
    // si capture == true : evenement est capture et n'est plus passé
    listen:function (e, trgt, callback, capture, c ,t){ // c et t = current et total images chargées pour manageEventsImageLoader
        e = e || window.event;
        // possibilite : capture = {passive:true}
        if(capture === true){
            if(e.preventDefault){
                e.preventDefault();
                e.stopPropagation();
            }else{
                e.returnValue = false;
                e.cancelBubble = true;
            }
        }
        var target = e.target || e.srcElement;
        if (callback) callback(e, trgt, target, c, t);
    },
    listenerAdd: function(id, evt, callback, capture, c, t){
        var elem = id !== window && id !== document && (id != '[object DocumentFragment]' && id != '[object HTMLImageElement]' && Object.prototype.toString.call(id) != '[object HTMLAnchorElement]' && Object.prototype.toString.call(id) != '[object HTMLLIElement]') ? this.$dc(id) : id;
        try{
            elem.addEventListener = elem.addEventListener || function (e, f) { elem.attachEvent('on' + e, f); };
            //manageEvents.myevents._addEvt({id:id, evt:evt, event:callback});
            elem.addEventListener(evt, function _events(e){
                manageEvents.myevents._addEvt({id:id, evt:evt, event:arguments.callee});
                // manageEvents.myevents._addEvt({id:id, evt:evt, event:callback});
                manageEvents.listen(e, this, callback, capture, c, t);
            },capture);
            elem.removeEventListener = elem.removeEventListener || function (e, f) { elem.detachEvent('on' + e, f); };
        }catch(e){
            console.log('%cERREUR : id "' + id + '" introuvable','color:#ff4e00;font-weight:bold');
            console.log(e);
            console.log('%c---------------------------','color:#ff4e00');
        }
       
    },
    listenerRemove: function (id, evt, cpt){
        var curEvt = this.myevents._remEvt({id:id, evt:evt});
        var elem = id !== window && id !== document ? this.$dc(id) : id;
        if (curEvt) {
            try{
                if(cpt == true){
                    elem.removeEventListener(evt, curEvt, true);
                }else{
                    elem.removeEventListener(evt, curEvt, false);
                }
                if(cpt === undefined) elem.removeEventListener(evt, curEvt);
            }catch(e){
                console.log('%cERREUR : id "' + id + '" introuvable','color:#ff4e00');
                console.log(e);
                console.log('%c---------------------------','color:#ff4e00');
            }
        }
    },
    forceRemove: function(id, evt, func, cpt){
        var elem = id !== window && id !== document ? this.$dc(id) : id;
        try{
            if(cpt == true){
                elem.removeEventListener(evt, func, true);
            }else{
                elem.removeEventListener(evt, func, false);
            }
            if(cpt === undefined) elem.removeEventListener(evt, func);
        }catch(e){
            console.log('%cERREUR : id "' + id + '" introuvable','color:#ff4e00');
            console.log(e);
            console.log('%c---------------------------','color:#ff4e00');
        }
    },
    addAclass: function (id, classe){
        var node = typeof id === 'object' ? id : this.$dc(id);
        if (node) node.classList ?  node.classList.add(classe) :  node.className += ' '+classe;
    },
    removeAclass: function(id,classe){
        var node = typeof id === 'object' ? id : this.$dc(id);
       // if (typeof this.$dc(id).classList.remove === 'function'){
        if(node){
            if (node.classList){
                node.classList.remove(classe);
            }else{
                node.className =  node.className.replace(' ' + classe, '').replace(classe, '');
            }
        }
    },
    hasAclass: function(id, cls) {
        var node = typeof id === 'object' ? id : this.$dc(id);
        if (node) return (' ' + node.className + ' ').indexOf(' ' + cls + ' ') > -1;
    }
}


if (!Array.prototype._addEvt){
    Array.prototype._addEvt = function(o){
        res = false;
        for(var t in this){
            if(this[t].id==o.id && this[t].evt==o.evt) res = true;
        }
        if(!res) this.push(o);
    }
}


if(!Array.prototype._remEvt){
    Array.prototype._remEvt = function(o){
        var res = false;
        var index = this._searchEvt(o);
        if(index != -1){
            var res = this[index];
            this.splice(index, 1);
        }
        if(typeof res === 'object'){
            return res.event;
        }else{
            return false;
        }
    }
}


if(!Array.prototype._searchEvt){
    Array.prototype._searchEvt = function(what, i) {
        i = i || 0;
        var L = this.length;
        while (i < L) {
            if(this[i].id === what.id && this[i].evt === what.evt) return i;
            ++i;
        }
        return -1;
    };
}
````

### plugin pour les Animation (CSS)

````js
/*
function objToString (obj) {
    var str = '';
    for (var p in obj) {
        //if (obj.hasOwnProperty(p)) {
            str += p + '::' + obj[p] + '\n';
        //}
    }
    return str;
}
*/



manageEvents = typeof manageEvents !== undefined ? manageEvents : {};

// Events :  animationstart / animationend / animationiteration
    
// standard : animationstart, animationiteration, animationend
// Firefox : animationstart, animationiteration, animationend
// webkit : webkitAnimationStart, webkitAnimationIteration, webkitAnimationEnd
// opera : oAnimationStart, oAnimationIteration, oAnimationEnd
// opera ? : oanimationstart, oanimationiteration, oanimationend
// IE10 : MSAnimationStart, MSAnimationIteration, MSAnimationEnd

manageEvents.versionAnimEvents = {
    animEvents : {
      'WebkitAnimation' : 'webkitanimation',
      'MozAnimation'    : 'animation',
      'OAnimation'      : 'oanimation',
      'oanimation'      : 'oanimation',
      'msAnimation'     : 'msanimation',
      'animation'       : 'animation'
    },
    nameEvents : {
        animationstart : {
            webkitanimation : 'webkitAnimationStart',
            oanimation : 'oAnimationStart oanimationstart',
            msanimation : 'MSAnimationStart',
            animation : 'animationstart'
        },
        animationend : {
            webkitanimation : 'webkitAnimationEnd',
            oanimation : 'oAnimationEnd oanimationend',
            msanimation : 'MSAnimationEnd',
            animation : 'animationend'
        },
        animationiteration : {
            webkitanimation : 'webkitAnimationIteration',
            oanimation : 'oAnimationIteration oanimationiteration',
            msanimation : 'MSAnimationIteration',
            animation : 'animationiteration'
        }
    },
    currentAnimEvents:{}, 
    init:function(){
        var el = document.createElement('bootstrap');
        var evtname='';
        //console.log(objToString(el.style));
        for (var name in this.animEvents) {
            //var lowername = this.animEvents[name];
            if (el.style[name] !== undefined) {
                evtname = this.animEvents[name];
            }
        }
        this.currentAnimEvents = {
            animationstart:this.nameEvents['animationstart'][evtname],
            animationend:this.nameEvents['animationend'][evtname],
            animationiteration:this.nameEvents['animationiteration'][evtname]
        }
        
        //console.log(this.currentAnimEvents);
        //console.log(Object.keys(this.currentAnimEvents).length);
        
    }
};

manageEvents.listenerAnimAdd=function(id, evt, callback, capture){
    var evta;
    for (var name in this.versionAnimEvents.currentAnimEvents){
        if (name == evt) evta = this.versionAnimEvents.currentAnimEvents[name];
    }
    if(evta === undefined){
        console.log(evt + ' / evenement inconnu pour le navigateur !');
        return;
    }
    try{
        this.$dc(id).addEventListener = this.$dc(id).addEventListener || function (e, f) { manageEvents.$dc(id).attachEvent('on' + e, f); };
        this.$dc(id).addEventListener(evta, function _events(e){
            manageEvents.myevents._addEvt({id:id, evt:evta, event:arguments.callee});
            manageEvents.listen(e,this, callback, capture);        
        },false);    
        this.$dc(id).removeEventListener = this.$dc(id).removeEventListener || function (e, f) { manageEvents.$dc(id).detachEvent('on' + e, f); };
    }catch(e){
        console.log('%cERREUR : id "' + id + '" introuvable','color:#ff4e00;font-weight:bold');
        console.log(e);
        console.log('%c---------------------------','color:#ff4e00');
    }
}

manageEvents.versionAnimEvents.init();
````

### plungin Transition (CSS)

````js
/*
function objToString (obj) {
    var str = '';
    for (var p in obj) {
        //if (obj.hasOwnProperty(p)) {
            str += p + '::' + obj[p] + '\n';
        //}
    }
    return str;
}
*/


manageEvents = typeof manageEvents !== undefined ? manageEvents : {};

// Events :  transitionend

// callback : e.elapsedTime / e.propertyName

manageEvents.versionTransEvents = {
    transEvents : {
      'WebkitTransition' : 'webkittransition',
      'MozTransition'    : 'moztransition',
      'OTransition'      : 'otransition',
      'transition'       : 'transition'
    },
    nameEvents : {
        transitionend : {
            webkittransition    : 'webkitTransitionEnd',
            oanimation          : 'oTransitionEnd ',
            moztransition       : 'mozTransitionEnd',
            transition          : 'transitionend'
        }

    },
    currentTransEvents:{},
    init:function(){
        var el = document.createElement('bootstrap');
        var evtname='';
        //console.log(objToString(el.style));
        
        for (var name in this.transEvents) {
            if (el.style[name] !== undefined) {
                evtname = this.transEvents[name];
            }
        }
        this.currentTransEvents = {        
            transitionend:this.nameEvents['transitionend'][evtname],
        }
        
    }
};

manageEvents.listenertransAdd=function(id, evt, callback, capture){
    var evta;
    //console.log(this.versionTransEvents.currentTransEvents);
    for (var name in this.versionTransEvents.currentTransEvents){
        if (name == evt) evta = this.versionTransEvents.currentTransEvents[name];
    }
    if(evta === undefined){
        console.log('%c'+evt + ' / evenement inconnu pour le navigateur !','color:#00b41c;font-weight:bold');
        return;
    }
    try{
        this.$dc(id).addEventListener = this.$dc(id).addEventListener || function (e, f) { manageEvents.$dc(id).attachEvent('on' + e, f); };
        this.$dc(id).addEventListener(evta, function _events(e){
            manageEvents.myevents._addEvt({id:id, evt:evta, event:arguments.callee});
            manageEvents.listen(e,this, callback, capture);        
        },false);    
        this.$dc(id).removeEventListener = this.$dc(id).removeEventListener || function (e, f) { manageEvents.$dc(id).detachEvent('on' + e, f); };
    }catch(e){
        console.log('%cERREUR : id "' + id + '" introuvable','color:#ff4e00;font-weight:bold');
        console.log(e);
        console.log('%c---------------------------','color:#ff4e00');
    }
}


manageEvents.versionTransEvents.init();
````

plugin pour video (cuepoints)

````js
manageEvents = typeof manageEvents !== undefined ? manageEvents : {};

manageEvents.CuePoint = {
    timeLag:.5,
    CuePoints:[],

    setTimeLag:function(t){
      this.timeLag = t;
    },

    executeCallBack:function(callBack,t,e,c,funct){
        if(t.currentTime < c) this.setCuePointNotPassed(c);
        if(t.currentTime >= c){
            if(this.getCuepointStatus(c) == 0) callBack(c,t,e);
            this.setCuePointPassed(c);
        }
    },

    callbackCP:function(e,c,funct){
        var target = e.target || e.srcElement;
        manageEvents.CuePoint.executeCallBack(function(c,t,e){
            funct(c,t);
        },target,e,c,funct);
    },

    pushCuePoint:function(c){
        this.CuePoints.push({status:0,timecp:c});
    },

    addCuePoints:function(v,c,funct){ // v = id video | c = cuepoint time | funct = hanlder final
        manageEvents.listenerAdd(v,'timeupdate', function(e){manageEvents.CuePoint.callbackCP(e,c,funct)},true);
        this.pushCuePoint(c);
    },

    getCuepointStatus: function(c){
        var ret = -1;
        for(var t = 0; t < this.CuePoints.length; t++){
            if (this.CuePoints[t].timecp == c) ret = this.CuePoints[t].status;
        }
        return ret;
    },

    setCuePointPassed: function(c){
        for(var t = 0; t < this.CuePoints.length; t++){
            if(this.CuePoints[t].timecp == c) this.CuePoints[t].status=1;
        }
    },

    setCuePointNotPassed: function(c){
        for(var t = 0; t < this.CuePoints.length; t++){
            if(this.CuePoints[t].timecp == c) this.CuePoints[t].status=0;
        }
    },

    reinitCuePointBefore: function(c){
        for(var t = 0; t < this.CuePoints.length; t++){
            if(this.CuePoints[t].timecp < c) this.CuePoints[t].status=0;
        }
    },

    setCuePointPassedBefore: function(c){
        for(var t = 0; t < this.CuePoints.length; t++){
            if(this.CuePoints[t].timecp < c) this.CuePoints[t].status=1;
        }
    },

    setCuePointNotPassedAfter: function(c){
        for(var t = 0; t < this.CuePoints.length; t++){
            if(this.CuePoints[t].timecp > c) this.CuePoints[t].status = 0;
        }
    },

    reinitAllCuePoints: function(){
        for(var t = 0; t < this.CuePoints.length; t++){
            this.CuePoints[t].status=0;
        }
    },

    gotoCuePoint: function(c, v, action){
        var vid = manageEvents.$dc(v);
        //console.log(this.CuePoints);
        if(this.getCuepointStatus(c) > -1) {

            switch(action){
                case 'play':
                    manageEvents.listenerAdd(v,'seeked', function(e){
                        var target = e.target || e.srcElement;
                        target.play();
                        manageEvents.listenerRemove(v,'seeked');
                    },true);
                    break;
                case 'stop':
                    manageEvents.listenerAdd(v,'seeked', function(e){
                        var target = e.target || e.srcElement;
                        target.pause();
                        manageEvents.listenerRemove(v,'seeked');
                    },true);
                    break;
            }

            this.setCuePointNotPassed(c);
            vid.currentTime = c;
            this.setCuePointPassedBefore(c);
            this.setCuePointNotPassedAfter(c);
        }else{
            console.log('erreur : cuepoint ' + c + ' inexistant');
        }
    },

    onCuePoints:function (v,c,callback){
        manageEvents.listenerAdd(v,'timeupdate',function(e){
            var target = e.target || e.srcElement;
            var curr = target.currentTime;
            var diff = Math.abs(c - curr);
            if(diff < manageEvents.CuePoint.timeLag && curr > c){
                callback(c,target);
            }
        },true)
    },

    onTimeUpdate:function(c,callback){}


}
````

### plugin pour les promises (basé sur Q) principalement utilisé pour les requêtes XHR

````js
manageEvents = typeof manageEvents !== undefined ? manageEvents : {};

try{
    if (typeof Q === 'function'){

        manageEvents = typeof manageEvents !== undefined ? manageEvents : {};

        manageEvents.promises =  Q;

        manageEvents.promises.request = false;

        // HTMLHttpRequest --------------------------------------------------------------------------------------------------------------------------

        manageEvents.promises.httpRequest = function(path, method, data, timeout, contentype, responsetype, accept){
            //console.log('request : ' + this.request);
            var deferred = this.defer();
            var responsetype = responsetype;
            
            // console.log('request', this.request)
            if(this.request){
                deferred.reject();
                this.request.abort();
                return deferred.promise;
            }else{
                this.request = new XMLHttpRequest(); // ActiveX blah blah
                this.request.open(method, path, true);
                if(responsetype) this.request.responseType = responsetype;
                this.request.setRequestHeader("X-Requested-With", "XMLHttpRequest");
                // this.request.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
                if(contentype) this.request.setRequestHeader("Content-type", contentype);
                if(accept) this.request.setRequestHeader("Accept", accept);
                this.request.onprogress = function(e){
                    deferred.notify(e.loaded/e.total);
                };

                this.request.onreadystatechange = function () {
                    if (this.readyState === 4) {
                        if (this.status === 200) {
                            if(responsetype == 'arraybuffer'){
                                deferred.resolve(this);
                           }else{
                               deferred.resolve(this.responseText);
                           }
                            manageEvents.promises.request = false; // detruit la requete lorsqu'elle aboutie
                        } else {
                            deferred.reject("HTTP " + this.status + " for " + path);
                        }
                    }
                };

                timeout && setTimeout(function(){deferred.reject('timeout expired'); manageEvents.promises.request = false;}, timeout);
                this.request.send(data);
                return deferred.promise;
            }
        };

        // FIN HTMLHttpRequest -----------------------------------------------------------------------------------------------------------------------

        manageEvents.promises.selectionne = function(id){
            var deferred = this.defer();
            var elem = manageEvents.$dc(id);
            var _targ = 0;
            //console.log(_targ);
            elem.onchange = function(e){
                //console.log(_targ);
                e = e || window.event;
                var _target = e.target || e.srcElement;
                _targ = _target.value;
                //console.log(_target.value);
                //deferred.resolve(_target.value);  // exécuté UNE seule fois
                deferred.notify(_target.value);
            };
            return deferred.promise;
        };

        manageEvents.promises.animation = function(id){
            var deferred = this.defer();
            var elem = manageEvents.$dc(id);
        };

        manageEvents.promises.aborting = function(){
            var deferred = this.defer();
            if(this.request){
                deferred.reject();
                this.request.abort();
                return deferred.promise;
            }
            return deferred.promise;
        };
    }else{
        throw 'Q promises introuvable !';
    }
}catch(err){
    console.log('Erreur : ' + err);
}
````
