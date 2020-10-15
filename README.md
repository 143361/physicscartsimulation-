<!DOCTYPE html>
<html> 
  <head>
    <title>Processing.JS inside Webpages: Template</title> 
  </head>
  <body>
	<!--This draws the canvas on the webpage -->
    <canvas id="mycanvas"></canvas> 
  </body>

  <!-- See https://khanacademy.zendesk.com/hc/en-us/articles/202260404-What-parts-of-ProcessingJS-does-Khan-Academy-support- for adding khan academy code -->
  <script src="https://cdn.jsdelivr.net/processing.js/1.4.8/processing.min.js"></script> 
  <script>
  var programCode = function(processingInstance) {
    with (processingInstance) {
      size(840, 600); //Canvas size
      frameRate(30);
    
    //PASTE CODE HERE 
      background(250, 202, 100);
      strokeWeight(0.0001); 
      fill(161, 228, 255);     
      rect(440,0,400,600); 

     //TITLE
         fill(0,0,0); 
         textSize(18); 
         text("CART ON AN INCLINE SIMULATION", 45, 22); 
        
    //Setting font
        var f = createFont("Helvetica Bold");
        textFont(f); 

     /***************************GLOBAL VARIABLES***********************/
     
     //Angle mode setting
        var angleSetting = 0.01745329251; 
     
     //Independant variables: 
         var mass = 0.5;
         var rampAngle = 20; 
         var cartAngle = 0;
         
     //Kinematics variables
         var surfaceArea = 1; 
         var cD = 1/500; 
         var g = 9.81; 
        
         var cartVel = 0; 
         var cartAccel = 0; 
         var changeInDistance = 0; 
         var changeInX = 0; 
         var changeInY =  0; 
         var airDrag = 0; 
         var forceDownSlope = 0; 
         
         var bounced = false; 
         var bouncedTime = 0; 
         var howManyBounce = 0; 
         
     //Sprites
         var rampY = 0; 
         var rampX = 0; 
         var rampLength = 252;
            
         var barrierX = 305; 
         var barrierY = 255; 
         var groundColour = (20, 20, 20); 
  
     //Time variables 
         var time = 0; 
         var timeStatus = 0; 
         var timerSecond = 0; 
         var timerMsecond = 0; 
         var startTime = 0; 
         var waitSomeTime = 0; 
         var timeNow = 0; 
     
     //Recording trials 
         var record = false; 
         var timePassed = 20; 
         var graphX = 215; 
         var graphY = 300; 
         var erase = false; 
     
    //Instructions 
        var help = false; 
        var showHelp = 1; 
             
         var ix = 820; 
         var iy = 580; 
         var iw = 20; 
         var ih = 20; 
          
    // Table
         var k = 0;  
         var show = false; 
     
        
/********************************TIME*********************************/ 

var timer = function() {
//If play is pressed:
    if (timeStatus === 1) {
        time++; //Use the time variable to measure time passed since play was clicked
        
        //On my computer, this makes time count in seconds and milliseconds
        timerSecond = Math.floor((time/30)); 
        
        timerMsecond = Math.floor(((time/30) - timerSecond) * 1000); 
    
    //Stop timing after a minute
      if (timerSecond > 59) {
            timeStatus = 0; //pause
            timerSecond = 0; //Restart seconds
            timerMsecond = 0; //Restart milliseconds
        }
        
    } 
}; 

/************************SLIDERS*************************/

            textSize(12); //Text size for sliders 
        
        //Slider object 
        var Slider = function(x, y, width, minV, maxV, currentV) { 
        //X and Y position and width 
         this.x = x; 
         this.y = y; 
         this.width = width; 
        
    
        //Minimum slider value
         this.min = minV;
        //Maximum slider value
         this.max = maxV;
        //Determining the appropriate ratio 
         this.ratio = (maxV - minV)/width; 
         this.currentValue = currentV || minV;  
         this.xPosition = this.x + (this.currentValue - this.min)/this.ratio; 
         
         //If the user's pointer is on the slider or not 
         this.onSlider = false; 
         
         //Drawing the slider:
         this.draw = function() {
            //Drawing the line using inputs 
            stroke(0, 0, 0);
            strokeWeight(3);
            line(this.x, this.y, this.width + this.x, this.y);
            
            //Drawing the 'draggable' ellipse 
            strokeWeight(0);
            stroke(0, 0, 0); 
            fill(255, 255, 255); 
            ellipse(this.xPosition, this.y, 20, 20); 
         }; 
         
         //If the user's pointer is on the slider
         this.touchingSlider = function() {
             //The slider may only be moved if it is within the minimum and maximum bounds
             //Inbuilt dist() function calculates distance between the pointer and maximum
           this.onSlider = dist(mouseX, mouseY, this.xPosition, this.y) <  20; 
         }; 
         
         //If the slider is dragged
             this.drag = function() {
                 //If pointer touching slider
                 if (this.onSlider) {
                     //chosenX is X position the slider is dragged to
                     //constrain() ensures slider remains within limits
                     var chosenX = constrain(mouseX, this.x, this.x + this.width);

                     //Slider X position equal to X position slider is dragged to
                     this.xPosition = chosenX; 

                     //Slider is being dragged: 
                     return true; 
                 } 
             }; 

         };     
         
        //List of all sliders  
        var allSlidersList = []; 
        
        //Defines slider properties into a list
        var makeSliders = function () {
        //X and Y starting positions, width
           var xbegin = 25; 
           var ybegin = 430; 
           var width = 120; 
           
         //Slider position when program is launched
           var defaultPosition = [mass,rampAngle,surfaceArea]; 
         
         //Minimum and maximum slider values
           var allMin = [0.2,0,0.1]; 
           var allMax = [1,90,2]; 
           
         //Create as many new sliders as are defined in the list of sliders
           for (var i = 0; i < defaultPosition.length; i++) {
               allSlidersList.push(new Slider(xbegin, ybegin, width, allMin[i], allMax[i], defaultPosition[i])); 
               
               //Move the position of the next slider down
                    ybegin = ybegin + 60; 
           }
           
           //Return value of allSlidersList to be accessed by other subroutines 
            return allSlidersList;
          
        };
        
        //Enter slider objects into the list of sliders (make properties accessible) 
        allSlidersList = makeSliders(); 
    
        //Draw sliders to screen 
        var drawGUI = function() {
            //for loop to iterate through the list of sliders
         for (var i = 0; i < allSlidersList.length; i++) {
             
             //Access current slider object by assigning it to a variable
             var s = allSlidersList[i]; 
             
             //Write the value of the slider to the screen
             fill(0,0,0);
             text((s.xPosition * s.ratio - s.x*s.ratio), s.x + s.width + 20, s.y + 10); 
             s.draw();
             
                 //Update variable values according to slider position 
                 if (i === 0) {
                        mass = (s.xPosition * s.ratio - s.x*s.ratio); 
                    } 
             
                     else if (i === 1) { 
                        rampAngle = (s.xPosition * s.ratio - s.x*s.ratio)*angleSetting;    
                    } 
             
                    else if (i === 2) { 
                      surfaceArea = (s.xPosition * s.ratio - s.x*s.ratio);  
                    } 
             
                }
                
        }; 
        
        //Pre-defined mouse-interaction functions relevant to sliders 
        mousePressed = function() {
    for (var k = 0; k < allSlidersList.length; k++) {
        allSlidersList[k].touchingSlider();
    }
  };
    
        mouseReleased = function() { 
    for (var k = 0; k < allSlidersList.length; k++) {
        allSlidersList[k].onSlider = false; 
    }
  }; 
  
        mouseDragged = function() {
    for (var i = 0; i < allSlidersList.length; i++) {
        if (allSlidersList[i].drag()) {
        }
    }
};

/************************ PAUSE, PLAY, RESTART BUTTONS **************************/ 
   
       var playButtonX = 0; 
       var playButtonY = 0; 
       var playButtonWidth = 0; 
       var playButtonHeight = 0; 
   
        //To draw the play button 
           var playButton = function(x, y, width, height) {
             var buttonX = x; 
             var buttonY = y; 
             var buttonHeight = height; 
             var buttonWidth = width; 
                 fill(3, 171, 255);
                 rect(buttonX, buttonY, buttonWidth, buttonHeight); 
                 fill(255,255,255);
                 triangle(buttonX+(buttonWidth/4), buttonY+(buttonHeight/4), buttonX+(buttonWidth/4), buttonY+(buttonHeight/4*3), buttonX+(buttonWidth/4*3), buttonY+(buttonHeight/2)); 

              playButtonX = buttonX; 
              playButtonY = buttonY; 
              playButtonHeight = buttonHeight; 
              playButtonWidth = buttonWidth; 

           }; 

        var pauseButtonX = 0; 
        var pauseButtonY = 0; 
        var pauseButtonHeight= 0; 
        var pauseButtonWidth = 0; 

        //To draw the pause button 
             var pauseButton = function(x, y, width, height) {
                 var buttonX = x; 
                 var buttonY = y; 
                 var buttonHeight = height; 
                 var buttonWidth = width; 
                     fill(3, 171, 255);
                     rect(buttonX, buttonY, buttonWidth, buttonHeight); 
                     fill(255,255,255);
                     rect(buttonX+(buttonWidth/4), buttonY+(buttonHeight/5), buttonWidth/6, buttonHeight/5*3); 
                      rect(buttonX+(buttonWidth/4*2.3), buttonY+(buttonHeight/5), buttonWidth/6, buttonHeight/5*3); 

                pauseButtonX = buttonX; 
                pauseButtonY = buttonY; 
                pauseButtonWidth = buttonWidth; 
                pauseButtonHeight = buttonHeight; 

           }; 

   var restartButtonX = 0; 
   var restartButtonY = 0;
   var restartButtonWidth = 0;
   var restartButtonHeight = 0; 
   
        //To draw the restart button 
        var restartButton = function(x, y, width, height) {
             var buttonX = x; 
             var buttonY = y; 
             var buttonHeight = height; 
             var buttonWidth = width; 
               fill(3, 171, 255);
               rect(buttonX, buttonY, buttonWidth, buttonHeight); 
               stroke(255, 255, 255);
               arc(buttonX+23, buttonY+24, buttonWidth+-18, buttonHeight+-22, -(3.14/2), 3.14); 
               stroke(255, 255, 255);
               fill(255,255,255);
               triangle(buttonX+17, buttonY+12, buttonX+26, buttonY+18, buttonX+26, buttonY+7);  

           restartButtonX = buttonX; 
           restartButtonY = buttonY; 
           restartButtonWidth = buttonWidth; 
           restartButtonHeight = buttonHeight; 
       }; 

/*************************** DRAW SPRITES *********************************/
    
    //Draw ramp
        var drawRamp = function() { 
         var RampX = 300; 
         var RampY = 290; 
         rampY = RampY;
         rampX = RampX;
         var RampWidth = 10; 
         fill(212, 85, 21);
            pushMatrix(); 
                translate(RampX, RampY); 
                rotate(rampAngle); 
                rect(0, 0, rampLength*-1, RampWidth*-1); 
            popMatrix(); 
        }; 

      //Draw cart
    var cartStartX = 0;
    var cartStartY = 0; 
    var cartX = 0; 
    var cartY = 0; 
    var cartWidth = 40; 
    var cartHeight = 17;

         var drawCart = function(X, Y) {
             stroke(0, 0, 0);
             strokeWeight(2); 
             
            //Body using predefined rectangle-drawing functions
                fill(158, 114, 33);
                rect(X, Y, cartWidth, cartHeight); //Predefined rectangle function

             //Wheels using predefined circle drawing functions
             ellipse(X+ cartWidth - (cartWidth/4), Y + cartHeight, cartWidth/3.5, cartWidth/3.5); 
             ellipse(X + (cartWidth/4), Y + cartHeight, cartWidth/3.5, cartWidth/3.5);

            //Motion sensor
                fill(255, 89, 89);
                rect(X + cartWidth/4, Y - 5, cartWidth/2, cartHeight/2);
         }; 
        
     
        //Function to change cart angle:  
             var changeCartAngle = function(angle) { 
                 fill(0, 0, 0);
                 pushMatrix(); //Save current coordinate system 
                     // Move the origin to the pivot point
                     translate(cartStartX, cartStartY); 
                     // Then pivot the grid
                       rotate(angle);
                       // Move the cart along
                       translate(cartX, cartY); 
                    //Draw the cart: 
                       drawCart(0,0); 
                 popMatrix(); //restore previous coordinate system

             };  

    var standX = 0; 
    var clampY = 0; 
    var standHeight = -220; 

    //Draw stand
     var drawStand = function() { 
         standX = rampX - cos(rampAngle)*(rampLength-20); 
         var standY = rampY;
         noStroke(); 
         fill(129, 120, 135);
         rect(standX, standY, 5, standHeight); 
         rect(standX-30, standY, 66, 5); 

         //Draw the clamp
             var clampWidth = 30;
             var clampDepth = 7; 
             var clampX = standX - clampWidth; 
             clampY = standY - tan(rampAngle)*(rampX - standX) - 7;
             fill(0,0,0);
             rect(clampX, clampY, clampWidth, clampDepth); 

         //Clamp hands
             pushMatrix(); 
             translate(clampX+clampWidth, clampY-clampDepth); 
             rotate(rampAngle); 
             rect(0, 0, 33, 16); 
             rect(0+11,-8,10,8); 
             rect(0+11,16,10,8); 
             popMatrix(); 

    }; 
    
    //Draw data logger
    var drawDataLogger = function() {
        var dLX = rampX - cos(rampAngle)*rampLength; 
        var dLY = rampY - sin(rampAngle)*rampLength; 
        
         pushMatrix(); 
         translate(dLX-24, dLY-36); 
         rotate(rampAngle); 
         fill(0, 255, 217);
         triangle(28,20, 4,7, 28,+2);  
         fill(255,0,0);
         rect(0, 0, 20, 20); 
         popMatrix(); 
         
    }; 
    
    //Draw barrier
    var drawBarrier = function() {
        fill(120, 61, 61);
        rect(barrierX, barrierY, 34, 35); 
    }; 
    

/**************************KINEMATICS EQUATION****************************/
        
var scalingRatio = 0.99; //Fits values to screen 

        var cartPlay = function() {
            //Rotate cart to the same angle as the ramp 
             cartAngle = rampAngle; 
        
                    if (bounced === false) { //Cart hasn't touched the barrier
                    
                    airDrag = cD * ((1.225*cartVel*cartVel)/2) * surfaceArea;
                   
                        //If cart hasn't stopped moving 
                    if (cartVel > 0)  {
                      forceDownSlope = mass*g*sin(rampAngle) - airDrag; 
                    } else { 
                      forceDownSlope = mass*g*sin(rampAngle) + airDrag; 
                    } 
                        
                        //Kinematics equations: 
                    cartAccel = forceDownSlope/mass; 
                    changeInDistance = cartVel*scalingRatio + 0.5*cartAccel;
                    changeInX = changeInDistance;
                    cartX = cartX + (changeInX*scalingRatio); 
                    
                        //Update cart velocity 
                    cartVel = cartVel + cartAccel;
                    
                    //If the cart hits the barrier 
                        if (cartX > 200) {
                            cartVel = -1*(sqrt(0.55))*cartVel; 
                            howManyBounce ++; 
                            bounced = true;    
                        } 
                    } 
                    
                    //If the cart is rebounding
                    if (bounced === true) { 
                        if (howManyBounce < 7) { //stops the cart bouncing infinately 
                        bouncedTime = bouncedTime+1/33; 
                        
                            //Kinematics equations: 
                            airDrag = cD * ((1.225*cartVel*cartVel)/2) * surfaceArea; 
                            forceDownSlope = mass*g*sin(rampAngle) - airDrag; 
                            cartAccel = forceDownSlope/mass; 
                            changeInDistance = cartVel*scalingRatio + 0.5*cartAccel;
                            changeInX = changeInDistance;
                            cartX = cartX + (changeInX*scalingRatio);
                            cartVel = cartVel + cartAccel;
                            
                            if (cartVel >= 0) { 
                                bounced = false; 
                            } 
                            
                        } else { //if it has bounced more than seven times 
                            cartVel = 0; 
                            cartAccel = 0; 
                        }
                                
                    } 
           
}; 

/***********************************************************************/


/********************ERROR MESSAGES************************/ 
var errorBoxX = 10; 
var errorBoxY = 30; 

var errorMessage = function(message) { 
    fill(0,255,0); 
    rect(errorBoxX-10, errorBoxY-18, 300, 25);
    fill(0,0,0); 
    text(message, errorBoxX, errorBoxY); 
};


/***********************RECORD TRIALS***********************/ 
  var recButtonX = 300; 
  var recButtonY = 325; 
  var recButtonWidth = 85; 
  var recButtonHeight = 38; 
 
        //Draw the record button 
         var drawRecordButton = function () {
             strokeWeight(3); 
             fill(38, 166, 45);
             rect(recButtonX, recButtonY, recButtonWidth, recButtonHeight); 
             fill(255,255,255);
             textSize(14); 
             text("RECORD", recButtonX+12, recButtonY+15, recButtonWidth, recButtonHeight); 

         }; 
 
  var accButtonX = 500; 
  var accButtonY = 14; 
  var accButtonWidth = 139; 
  var accButtonHeight = 31;
  var displayAccelResults = false; 
 
        //ACCELERATION button 
     var accelResultsButton = function() {
         fill(255,255,255); 
         strokeWeight(3); 
         fill(0, 0, 255);
         rect(accButtonX, accButtonY, accButtonWidth, accButtonHeight); 
         fill(255,255,255);
         
         text("ACCELERATION", accButtonX+10, accButtonY+10, accButtonWidth, accButtonHeight); 
 }; 
 
  var velButtonX = 670; 
  var velButtonY = 14; 
  var velButtonWidth = 106; 
  var velButtonHeight = 31;
 
        //VELOCITY button 
     var velocityResultsButton = function() {
         strokeWeight(3); 
         fill(247, 35, 35);
         rect(velButtonX, velButtonY, velButtonWidth, velButtonHeight); 
         fill(255,255,255);
         text("VELOCITY", velButtonX+16, velButtonY+10, velButtonWidth, velButtonHeight); 
     }; 
    
     var graphXPosition = 505; 
     var graphYPosition = 100; 

     var accelResults = []; 
     var velResults = []; 
     var graphTime  = []; 
     var w = 0; 

     var accelDVRange = 200; 
     var accelIVRange = 300; 
     var velDVRange = 200; 
     var velIVRange = 300; 
    
     var maxAccel = 9.81*1.1; 
     var maxVel = 100; 
     var maxTime = (2); 
     
        //Graph background
        fill(247, 247, 247);
        rect(graphXPosition, graphYPosition, accelIVRange, accelDVRange); 

 //Drawing graphs 
 var drawResultsGraph = function() { 

    if (displayAccelResults === true) {
    
        strokeWeight(0.00001); 
        fill(161, 228, 255);
        rect(graphXPosition-50, graphYPosition-5, 50, accelDVRange+5); 
        fill(161, 228, 255);
        rect(graphXPosition-20, graphYPosition-30, accelIVRange+30, 30); 
     
            fill(0,0,0);
            textSize(15);
            text("ACCEL (ms^2) versus time (s)", graphXPosition+50, graphYPosition-11); 
            
             fill(0,0,0);
             text(-9.81, graphXPosition-50, graphYPosition + accelDVRange);
             
             text(0, graphXPosition-20, graphYPosition + accelDVRange/2); 
            
             text(9.81, graphXPosition-45, graphYPosition+10);
             
    } 
    
    if (displayAccelResults === false) { 
          
          //White rectangle to draw over text   
          strokeWeight(0.00001); 
          fill(161, 228, 255);
          rect(graphXPosition-50, graphYPosition-5, 50, velDVRange+5); 
          
          //Draw axis label
          fill(161, 228, 255);
          rect(graphXPosition-20, graphYPosition-30, accelIVRange+30, 30); 
          textSize(15); 
          fill(0,0,0);
          text("VELOCITY (ms^2) versus time (s)", graphXPosition+30, graphYPosition-11);
        
          //Draw maximum, minimum and mid range axis labels 
          fill(0,0,0);
             text(-100, graphXPosition-45, graphYPosition + velDVRange);
             
             text(0, graphXPosition-30, graphYPosition + velDVRange/2); 
            
             text(100, graphXPosition-40, graphYPosition + 10);      
    } 


     if (timeStatus === 1) { //Play has been clicked 
         
         if (record === true) { //Record has been clicked 
             if (displayAccelResults === true) { //ACCELERATION has been clicked

                    if (cartAccel > 0) { //If the cart is moving 

                        //Push acceleration and time to arrays
                         accelResults.push(cartAccel); 
                         graphTime.push(timerMsecond); 

                        //Draw the ellipse
                         fill(255, 0, 0);
                         strokeWeight(3); 
                         ellipse(graphXPosition+(((timerMsecond/1000)+timerSecond)/maxTime)*accelIVRange, ((graphYPosition+accelDVRange) - (accelDVRange/2) - ((cartAccel/maxAccel)*accelDVRange)), 1, 1); 
                         strokeWeight(0);     
                    } 
             } 

             if (displayAccelResults === false) { //VELOCITY has been clicked

                 if (cartAccel > 0) { //If the cart is moving 

                    //Push acceleration and time to arrays 
                     velResults.push(cartVel); 
                     graphTime.push(timerMsecond); 

                    //Draw the ellipse
                     strokeWeight(3); 
                     fill(255, 0, 0);
                     ellipse(graphXPosition+(((timerMsecond/1000)+timerSecond)/maxTime)*velIVRange, (graphYPosition+velDVRange) - (velDVRange/2) - (cartVel/maxVel)*velDVRange, 1, 1);  
                     strokeWeight(0); 
                 }     
             }     
        }
     }
 
 }; 
 
  var seeArraysButtonX = 0; 
  var seeArraysButtonY = 0; 
  var seeArraysButtonWidth = 0; 
  var seeArraysButtonHeight = 0;
 
 var showResultsButton = function (x,y,width,height) {
     strokeWeight(3); 
     fill(5, 138, 136);
     rect(x, y, width, height); 
     fill(255,255,255);
     textSize(14); 
     text("SHOW FULL RESULTS", x+7, y+19); 
     fill(255, 0, 255);
     fill(255, 0, 255);
 
     seeArraysButtonX = x; 
     seeArraysButtonY = y; 
     seeArraysButtonWidth = width; 
     seeArraysButtonHeight = height;
        
 }; 
 
 /*
 var tabAccButtonX = 0; 
 var tabAccButtonY = 0; 
 var tabAccButtonWidth = 0; 
 var tabAccButtonHeight = 0;
 
 
  var showAccTable = function (x,y,width,height) {
     strokeWeight(0); 
     fill(255, 162, 13);
     rect(x, y, width, height); 
     fill(255,255,255);
     text("ACCELERATION VALUES", x+5, y+10, width, height); 
     
     tabAccButtonX = x; 
     tabAccButtonY = y; 
     tabAccButtonWidth = width; 
     tabAccButtonHeight = height; 
    
 }; 
 */ 
 
 //VELOCITY GRAPH
 
 //VELOCITY TABLE   
 
 //CLEAR RESULTS BUTTON
 var eraButtonX = 690; 
 var eraButtonY = 340;
 var eraButtonWidth = 130; 
 var eraButtonHeight = 30; 
 
  var drawEraseButton = function () {
     fill(179, 23, 158);
     rect(eraButtonX-10, eraButtonY-20, eraButtonWidth, eraButtonHeight); 
     fill(255, 255, 255);
     text("ERASE GRAPH", eraButtonX, eraButtonY); 
     
    var stopErase = function() {
         erase = false; 
     }; 
     
     if (erase === true) { 
         accelResults = []; 
         graphTime = []; 
         fill(247,247,247);
         rect(graphXPosition, graphYPosition, accelIVRange, accelDVRange);
         stopErase(); 
     } 
     

 }; 
 
 var displayAccTable = false; 
 var displayVelTable = false; 
 
/***********************APPARTUS LIST*************************/
var apparatusListX = 215; 
var apparatusListY = 387; 
var apparatusListWidth = 200; 
var apparatusListHeight = 130; 

var apparatusList = function() { 
    fill(179, 226, 242);
    rect(apparatusListX, apparatusListY, apparatusListWidth, apparatusListHeight); 
    
    textSize(11)
    
    fill(0,0,0);
    text("APPARATUS LIST", apparatusListX+(apparatusListWidth*0.08), apparatusListY+(apparatusListHeight*0.2)); 
    
    text("Clamp stand", apparatusListX+(apparatusListWidth*0.08), apparatusListY+(apparatusListHeight*1.5*0.2)); 
    
    text("Cart with motion sensor", apparatusListX+(apparatusListWidth*0.08), apparatusListY+(apparatusListHeight*2*0.2)); 
    
     text("Data logger", apparatusListX+(apparatusListWidth*0.08), apparatusListY+(apparatusListHeight*2.5*0.2)); 
    
    text("Ramp (length: " + rampLength*0.0025 + "m)", apparatusListX+(apparatusListWidth*0.08), apparatusListY+(apparatusListHeight*3*0.2)); 
    
      text("Clamp (Height: " + round(((rampY - clampY)*0.0025)*100)/100 + "m)", apparatusListX+(apparatusListWidth*0.08), apparatusListY+(apparatusListHeight*3.5*0.2)); 
      
       text("Metal barrier", apparatusListX+(apparatusListWidth*0.08), apparatusListY+(apparatusListHeight*4*0.2)); 
    
}; 
 
/***************************************************************/ 
 
        
 /*******DRAW TABLE***********/ 
        
    var drawInstructions = function() { 
        fill(250,250,250);
         rect(500, 380, 300, 210);
        
         fill(0,0,0);
         textSize(15); 
         text("HOW TO USE THIS SIMULATION", 530, 410);
         textSize(10); 
         var IY = 430; 

         text("1) View the live simulation in the top left of", 520, IY)
         text(" the screen", 520, IY+20); 
         text("2) Use the sliders to change mass, ramp angle ", 520, IY+40); 
         text("and surface area", 520, IY + 60); 
         text("3) Click play, pause, and restart buttons to", 520, IY + 80); 
         text("control the simulation", 520, IY + 100);
         text("4) Click record and then 'Acceleration' or 'Velocity'", 520, IY + 120); 
         text("to view their respective graphs", 520, IY + 140)
    } 
        
       
    
 var drawAccelTable = function() {
      fill(0,0,0); 
         text("SAMPLE ACCELERATION MEASUREMENTS", 500, 400);

         strokeWeight(2); 
         line(495, 415, 495, 535); 
         line(555, 415, 555, 535);
         line(700, 415, 700, 535);

         line(495, 415, 700, 415);
         line(495, 435, 700, 435);
         line(495, 535, 700, 535);

         text("ACCELERATION (ms^2)", 560, 430);
            text(Math.floor(accelResults[0]), 560, 450); 
            text(Math.floor(accelResults[1]), 560, 470); 
            text(Math.floor(accelResults[2]), 560, 490); 
            text(Math.floor(accelResults[3]), 560, 510); 
            text(Math.floor(accelResults[4]), 560, 530); 


         text("TIME (s)", 500, 430); 
            text(Math.floor(graphTime[0]), 500, 450); 
            text(Math.floor(graphTime[1]), 500, 470); 
            text(Math.floor(graphTime[2]), 500, 490); 
            text(Math.floor(graphTime[3]), 500, 510); 
            text(Math.floor(graphTime[4]), 500, 530); 

         fill(10,10,10); 
         textSize(10); 
          text("If NaN displayed, you forgot to record acceleration!", 500, 555);
          text("Restart and this time make sure to click 'ACCCELERATION'", 500, 575);
          text("and 'RECORD' before playing...", 500, 585);  

 }; 
        
var drawVelocityTable = function() { 
     fill(0,0,0); 
     text("SAMPLE VELOCITY MEASUREMENTS", 500, 400);

     strokeWeight(2); 
     line(495, 415, 495, 535); 
     line(555, 415, 555, 535);
     line(650, 415, 650, 535);

     line(495, 415, 650, 415);
     line(495, 435, 650, 435);
     line(495, 535, 650, 535);

     text("VELOCITY" + "(ms)", 560, 430); 
        text(Math.floor(velResults[0]), 560, 450); 
        text(Math.floor(velResults[1]), 560, 470); 
        text(Math.floor(velResults[2]), 560, 490); 
        text(Math.floor(velResults[3]), 560, 510); 
        text(Math.floor(velResults[4]), 560, 530); 

    text("TIME (s)", 500, 430); 
        text(Math.floor(graphTime[0]), 500, 450); 
        text(Math.floor(graphTime[1]), 500, 470); 
        text(Math.floor(graphTime[2]), 500, 490); 
        text(Math.floor(graphTime[3]), 500, 510); 
        text(Math.floor(graphTime[4]), 500, 530); 

     fill(10,10,10); 
     textSize(10); 
      text("If NaN displayed, you forgot to record velocity!", 500, 555);
      text("Restart and this time make sure to click 'VELOCITY'", 500, 575);
      text("and 'RECORD' before playing...", 500, 585); 

};         
        
 /*********************/ 
 
 
/*****************************DRAW FUNCTION**********************************/ 
 var draw = function() {
     
 //Background colours 
     
    strokeWeight(0.0001);
    fill(255, 255, 255);
     
    fill(155, 250, 142); 
    rect(10, 30, 407, 282); 
    
    fill(179, 226, 242);  
    rect(10, 387, 192, 205);  
    
    //Draw sliders
    drawGUI(); 
             fill(0, 0, 0);
             text("MASS (kg)", 30, 410); 
             text("RAMP ANGLE (degrees)", 30, 470); 
             text("SURFACE AREA (m^2)", 30, 530); 
  

    //Draw buttons 
    playButton(30, 325, 46, 46); 
    pauseButton(100, 325, 46, 46); 
    restartButton(170, 325, 46, 46);
    drawRecordButton(); 
    
    //Draw sprites 
    drawBarrier(); 
    drawStand(); 
    drawDataLogger(); 
    drawRamp();
    
    //Apparatus list
    apparatusList(); 
    
    //Drawing cart at starting position 
    cartStartY = rampY - rampLength*sin(rampAngle)-(cartHeight+(cartWidth/7))*cos(rampAngle)-10;
    cartStartX = rampX - rampLength*cos(rampAngle)+(cartHeight+(cartWidth/7))*sin(rampAngle);
    
    timer(); 
    fill(207, 255, 220); 
    rect(270,55,135,40); 
    fill(0, 0, 0);
    textSize(15); 
    text("Time (s): " + timerSecond + ":" + timerMsecond, 290, 80);
    
    if (timeStatus === 0) { 
        //Drawn once in the loop (at the start when the user is adjusting the cart angle)
        changeCartAngle(rampAngle); 
    }
    
    if (timeStatus === 1) {
       cartPlay(); 
       //with changing X and Y positions according to kinematics equations  
    }

    changeCartAngle(rampAngle); 
    
    //RECORDING TRIAL RESULTS
        drawResultsGraph(); 
        accelResultsButton(); 
        velocityResultsButton(); 
        
        if (record === true) {
            fill(255, 0, 0);
            ellipse(recButtonX, recButtonY, 15, 15); 
        } 
        
        drawEraseButton(); 
        
    //Error messages: 
 if (timerSecond === 59) { 
     fill(0,0,0);
    errorMessage("You have been timing for over one minute.");  
    timeStatus = 0; 
}

if (rampAngle >= 80) { 
    fill(0, 0, 0);
    errorMessage("This angle will most probably not be feasible");  
}

if (key === 32) { 
    errorMessage(""); 
} 
 
    showResultsButton(500, 320, 164, 29);
   
     //Numerical velocity and acceleration values visible:
     strokeWeight(0.00001); 
     fill(179, 226, 242); 
     rect(215, 525, 200, 65);
     
     fill(0,0,0); 
     textSize(12); 
     text("VELOCITY: " + Math.floor(cartVel), 230, 547); 
     text("ACCELERATION: " + Math.floor(cartAccel), 230, 577);
     
    
     
     //Instructions

     
     fill(37, 27, 227); 
     ellipse(ix, iy, iw, ih); 
     fill(255,255,255); 
     text("i", ix, iy+2); 
    
     
     if (help === true) {    
     //INSTRUCTIONS 
         drawInstructions(); 
         
     } else if (help === false) { 
        fill(161, 228, 255);
        rect(490, 380, 320, 210);
     }
     
     
     
     //TABLE:
     if (show === true && help === false) { //Button clicked, instructions not in the way 
        
         fill(40,60, 70); 
         ellipse(seeArraysButtonX, seeArraysButtonY, 20, 20); 


          fill(0,0,0); 
             if (cartVel === 0) { //If cart has stopped moving 
                 if (displayAccelResults === true && timerSecond > 1) { 
                     drawAccelTable();  
                 } 
                 
                 if (displayAccelResults === false && timerSecond > 1) { 
                     drawVelocityTable(); 
                 }     
             }
     }
};



/*********************MOUSE INTERACTION****************************/

        //Predefined function called if the mouse is clicked: 
    mouseClicked = function() { 
        
        //if statement checking if the mouse is within the area of the button 
      if (mouseX >= playButtonX && mouseX <= playButtonX+playButtonWidth && mouseY >= playButtonY && mouseY <= playButtonY+playButtonHeight) {
          timeStatus = 1; //updates time status to play 
        }
        
      if (mouseX >= pauseButtonX && mouseX <= pauseButtonX+pauseButtonWidth && mouseY >= pauseButtonY && mouseY <= pauseButtonY+pauseButtonHeight) {
            timeStatus = -1; //updates time status to paused 
        }
        
     if (mouseX >= restartButtonX && mouseX <= restartButtonX+restartButtonWidth && mouseY >= restartButtonY && mouseY <= restartButtonY+restartButtonHeight) {
         //TIME
             timeStatus = 0; 
             time = 0; 
             timerSecond = 0; 
             timerMsecond = 0; 
         //GRAPH
             accelResults = []; 
             graphTime = []; 
             w = 0; 
             fill(247, 247, 247);
             rect(graphXPosition, graphYPosition, accelIVRange, accelDVRange); 
         //TABLE
             show = false;      
             strokeWeight(0.00001);
             fill(161, 228, 255); 
             rect(seeArraysButtonX-10, seeArraysButtonY-10, seeArraysButtonWidth+10, seeArraysButtonHeight+10); 
             
         //CART DRAWING 
             bounced = false; 
             howManyBounce = 0; 
             bouncedTime = 0; 
         
         //Cart back to start position, reset origin 
             cartX = 0; 
             cartY = 0; 
             cartStartY = rampY-sin(rampAngle)*rampLength-sin(180-(90+rampAngle))*cartWidth+10;
             cartStartX = rampX-rampLength+(rampLength-cos(rampAngle)*rampLength)+(cos(180-(90+rampAngle))*cartWidth); 
             cartVel = 0; 
         
        
     } 
     
      if (mouseX >= recButtonX && mouseX <= recButtonX+recButtonWidth && mouseY >= recButtonY && mouseY <= recButtonY+recButtonHeight) {
          record = true; 
      }
      
       if (mouseX >= eraButtonX && mouseX <= eraButtonX+eraButtonWidth && mouseY >= eraButtonY && mouseY <= eraButtonY+eraButtonHeight) {
          erase = true; 
      }
      
       if (mouseX >= velButtonX && mouseX <= velButtonX+velButtonWidth && mouseY >= velButtonY && mouseY <= velButtonY+velButtonHeight) {
          displayAccelResults = false; 
      }
      
       if (mouseX >= accButtonX && mouseX <= accButtonX+accButtonWidth && mouseY >= accButtonY && mouseY <= accButtonY+accButtonHeight) {
           displayAccelResults = true; 
      }
      
      if (mouseX >= seeArraysButtonX && mouseX <= seeArraysButtonX+seeArraysButtonWidth && mouseY >= seeArraysButtonY && mouseY <= seeArraysButtonY+seeArraysButtonHeight) {
           show = true; 
      }
     
     if (mouseX >= ix-iw && mouseX <= ix+iw*2 && mouseY >= iy-ih && mouseY <= iy+ih*2) {
       
        showHelp = showHelp + 1; 

     if (showHelp%2 === 0) {
         help = true; 
     } else { 
         help = false; 
     }


         
     } 
    
        
    }; 
        

    }};

  // Get the canvas that ProcessingJS will use
  var canvas = document.getElementById("mycanvas"); 
  // Pass the function to ProcessingJS constructor
  var processingInstance = new Processing(canvas, programCode); 
  </script>
</html>
