# Flappy-Bird
class Point {
    /**
     * Create a point.
     * @param {number} x - The x value.
     * @param {number} y - The y value.
     */
    constructor(x, y) {  
      // Fill in code to set attributes of point 
      this.myX = x;
      this.myY = y;
    }
    
       /**
     * Returns the distance between itself and another point
     * @param {Point} p the point to find the distance between
     * @return {number} distance between itself and p
     */
    
    distance(p) {
       return Math.sqrt((p.x-this.myX)**2 + (p.y-this.myY)**2);
    }

    get x() {
      return this.myX;
    }
    
    get y() {
      return this.myY;
    }
}    

/**
 * A Bird class maintains the position of a bird over time
 */
class Bird {
  
  /**
   * Create a new Bird.
   * @param {Point} startPosition - The 2D starting position of the Bird (x, y)
   * @param {number} startXSpeed - The starting horizontal speed of the bird (pixels/second)
   * @param {number} gravity - The change in the y velocity due to gravity (pixels/second)
   * @param {number} flapUpSpeed - The y velocity (opposite direction of gravity) caused by a flap
   */
  
  constructor(startPosition, startXSpeed, gravity, flapUpSpeed) {
    this.currentPosition = startPosition;
    this.currentXSpeed = startXSpeed;
    this.gravity_ = gravity;
    this.flapUpSpeed_ = -flapUpSpeed;
    this.currentYSpeed = 0;
    this.width_ = 34;
    this.height_ = 24;
  }
  
  /**
   * Updates the position of the bird (both x and y coordinates)
   * @param {number} secondsElapsed - the number of seconds that passed since the last move
   */
  
  move(secondsElapsed) {
    //update x position
    let newX = (this.currentXSpeed * secondsElapsed) + this.currentPosition.x;
    
    //update y position
    let newY = (this.currentYSpeed * secondsElapsed) + this.currentPosition.y;
    
    this.currentPosition = new Point(newX, newY);  
    
    //update y velocity
   this.currentYSpeed = (this.gravity_ * secondsElapsed) + this.currentYSpeed;
  }
  
  /**
   * Updates the bird's y velocity caused by a flap given by flapUpSpeed
   */
  
  flap() {
    //change y velocity
    this.currentYSpeed = this.flapUpSpeed_;

  }
  
  /**
   * @type {Point}
   */
  
  get position() {
    // getter for current position of Bird
    // return variable that holds bird's position
    return this.currentPosition;
  }
  get width() {
    return this.width_;
  }
  get height() {
    return this.height_;
  }
}

class Pipe {
  constructor (position) {
    this.position_ = position;
    this.gap_ = 150;
    this.width_ = 52; 
    this.height_ = 320;
  }
  
  get gap() {
    return this.gap_;
  }
  
  get position() {
    return this.position_;
  }
  
  get height() {
    return this.height_;
  }
}

class PipeWorldView {
  constructor(pw) {
    this.pw_ = pw;
    this.pipeBottomImage_ = new Image();
    this.pipeTopImage_ = new Image();
    this.pipeBottomImage_.src = "https://studio.code.org/blockly/media/skins/flappy/obstacle_bottom.png";
    this.pipeTopImage_.src = "https://studio.code.org/blockly/media/skins/flappy/obstacle_top.png";

  }
  
  render(gameContext, xPosOfBird) {
    let pipeTopX;
    let pipeTopY ;
    
    for(let i = 0; i<= 4; i++) {
      pipeTopX = this.pw_.getPipeByNumber(i).position.x;
      pipeTopY = this.pw_.getPipeByNumber(i).position.y;
      
      // top pipe
      gameContext.drawImage( this.pipeTopImage_, pipeTopX-xPosOfBird, pipeTopY,  this.pipeTopImage_.width,  this.pipeTopImage_.height);
      // bot pipe
      gameContext.drawImage( this.pipeBottomImage_, pipeTopX-xPosOfBird, (pipeTopY+ this.pipeTopImage_.height +this.pw_.getPipeByNumber(0).gap),  this.pipeBottomImage_.width,  this.pipeBottomImage_.height);
    }// for loop end
    
   } // render end
}  

class PipeWorld {
  constructor (){
    this.pipes_ = [];
    let currentPoint = new Point(350, Math.random()*-300);
    for(let i = 1; i<= 5; i++) {
      this.pipes_.push(new Pipe(currentPoint));
      currentPoint= new Point(currentPoint.x + 250, (Math.random()*-300));
    }
  }
  getPipeByNumber(n) {
    return this.pipes_[n];
  }
  
  getNumberOfPipes(){
    return this.pipes_.length;
    
  }
  
  shiftPipes() {
    this.pipes_.shift();
  }
  
  pushNewPipe(newXPos,newYPos) {
    this.pipes_.push(new Pipe(new Point(newXPos, newYPos)));
  }
  
  get position() {
    return this.currentPosition;
  }
 
}

class World {
  constructor () {
    
  // models
  //startPosition, startXSpeed, gravity, flapUpSpeed
    this.bird_ = new Bird(new Point(5, 5), 130, 150, 120);
    this.pw_ = new PipeWorld();
    
  }
  
  checkIfFirstPipeIsOffScreen() {
    // condition will check if first pipe's position is completely off screen
    if ((this.pw_.getPipeByNumber(0).position.x - this.bird_.position.x) < -52 ) {
      // document.getElementById("checkIfOffScreen").value = "YES";
    // SHIFT: shift all pipes to the left by 1 scu that the first pipe disappears
     this.pw_.shiftPipes();
    
    // PUSH: create new pipe at the last pipe's position + 250
    // Need to pass in new value for new pipe based on last pipe's position + 250
    this.pw_.pushNewPipe((this.pw_.getPipeByNumber(this.pw_.getNumberOfPipes()-1).position.x + 250), Math.random()*-300);
    
    }
    
  }
  
  // Collision
  checkForCollision() {
    /* ONLY CARE ABOUT FIRST PIPE: (true means bird hits the pipe)
      - If statement: CHECK if bird is near first pipe
        1. Check if (bird's position is less than or equal to pipe's x position ON CANVAS (pipe.x - bird.x) AND bird's x position < the pipe's x position + pipe's width)
          - If so, CHECK if the bird is NOT in the gap
            2. Check if (bird's position is less than top pipe's bottom-tip's y-coordinate OR if bird's position is greater than the bottom pipe's top-tip's y-coordinate)
              RETURN TRUE
        3. Else, RETURN FALSE
    */ 
      if ((this.bird_.position.x >= (this.pw_.getPipeByNumber(0).position.x - this.bird_.width)))  {
        // (this.bird_.position.x < (this.pw_.getPipeByNumber(0).position.x + this.pw_.getPipeByNumber(0).width))
          console.log("ENTERED GAP");
        
        if ((this.bird_.position.y < (this.pw_.getPipeByNumber(0).position.y + this.pw_.getPipeByNumber(0).height)) || (this.bird_.position.y > (this.pw_.getPipeByNumber(0).position.y + this.pw_.getPipeByNumber(0).gap + this.pw_.getPipeByNumber(0).height - this.bird_.height))) {
           return true; 
          
        }
      } 
        else {
           return false;
      }
    
    
  }
  
  get bird() {
    return this.bird_;
  }
  
  get pipeWorld() {
    return this.pw_;
  }
  
}

// class LoseScreen {
//
//}

class WorldView {
 
    constructor(world) {
      this.w_ = world;
      this.pw_ = this.w_.pipeWorld;
      this.bird_ = this.w_.bird;
      
      // make views
      this.pwv = new PipeWorldView(this.pw_);
      // this.ls = new LoseScreen();
      
      this.birdImage = new Image();  
      this.skyBackgroundImage = new Image();
      this.birdImage.src = "https://studio.code.org/blockly/media/skins/flappy/avatar.png";
      this.skyBackgroundImage.src = skyImageData; //"cloud-background.jpg";
      
      this.canvasElement = document.getElementById("game");
      this.gameContext = this.canvasElement.getContext("2d");
      this.canvasElement.width_ = window.innerWidth;
      this.canvasElement.hieght_ = window.innerHeight;
    }
 
    render() { 
      // reset canvas to blank
      this.gameContext.clearRect(0, 0, this.canvasElement.width, this.canvasElement.height); 
      
      // draw sky based on bird x position
      this.gameContext.drawImage(this.skyBackgroundImage, -(this.bird_.position.x) %(this.skyBackgroundImage.width - this.canvasElement.width), 0, this.skyBackgroundImage.width, this.skyBackgroundImage.height);
      
      // draw bird AFTER it's moved
      this.gameContext.drawImage(this.birdImage, 5, Math.trunc(this.bird_.position.y), this.birdImage.width, this.birdImage.height);
      
      // draw Pipes
      this.pwv.render(this.gameContext, this.bird_.position.x);
    }
}


class Controller {
  constructor() {
    this.w = new World();
    this.wv = new WorldView(this.w);
    window.addEventListener("click", this.w.bird.flap.bind(this.w.bird));
  }
  
  start() {
    this.lastTime = 0;
    
    let runGame = milliseconds => {
      // move everything/check on pipes
      if (this.w.checkForCollision()) {
        // lose screen with restart button
      } else {
        this.w.bird.move((milliseconds - this.lastTime)/1000);
        this.w.checkIfFirstPipeIsOffScreen();
        
        // render any changes made
        this.wv.render(); 
        this.lastTime = milliseconds;
        
        // Check for bugs
        document.getElementById("birdXPos").value = this.w.bird.position.x;
        document.getElementById("checkFirstPipe").value = this.w.pipeWorld.getPipeByNumber(0).position.x;
        document.getElementById("checkLastPipe").value = this.w.pipeWorld.getPipeByNumber(this.w.pipeWorld.getNumberOfPipes()-1).position.x;
      }
      
      // infinite loop
      requestAnimationFrame(runGame);  
    }

    requestAnimationFrame(runGame);
  } // start()
  
}

// START CODE HERE

let msToSec = milliseconds => milliseconds/1000;
let distance = (v, t) => v * t;

// startPosition, startXSpeed, gravity, flapUpSpeed
let c = new Controller();
c.start();

/* "circle to rectangle collision"
Bird: w=34, h=34
find center: width = diameter, so center is 1/2 of that
center = (Bird XPos + 1/2 width_, Bird XPos - 1/2 height_);
if (distance between point on rect & circle center is <= radius)
then collision --> restart game 
else
no collision continue game
distance function already in point class 
*/

/* //Bird
class Circle {
   constructor (radius, diameter, center, width, height) {
     this.diameter_ = this.width_;
     this.radius_ = (1/2 this.diameter_);
     this.center_ = center;
     this.width_ = 34;
     this.height_ = 34;
     let center_ = new Point((this.currentPosition.x + 1/2 width_), (this.currentPosition.x - 1/2 height_));
   }
}

Pipe
class Rectangle {
  constructor (position, width, height) {
    this.x = x;
    this.y = y;
    this.width_ = 52; 
    this.height_ = 320;
    
  }  
}

class CollisionDetector {
 // "Static method"
   constructor (distance) {
     this.distance_ = return Math.sqrt((p.x-this.myX)**2 + (p.y-this.myY)**2);
   }  
     if (this.distance <= this.radius_) {
   //collision detected 
  
 } 
   else {
 //no collision continue game
   requestAnimationFrame(runGame);
   };
 }
*/
