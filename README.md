import processing.video.*; 
import gohai.simpletweet.*;

//////////////////////////////////////////////////////////////////////////////
                        
PImage tweet;                        
import twitter4j.conf.*;
import twitter4j.*;
import twitter4j.auth.*;
import twitter4j.api.*;
import java.util.*;
Twitter twitter;
String searchString = "Bio Pixels";
ArrayList<Status> tweets;
int currentTweet;

//////////////////////////////////////////////////////////////////////////////

SimpleTweet scbasedPhoto; 
int videoScale = 4; 
int cols; 
int rows; 
Capture video; 
Movie movie; 
Differentiation D; 

//////////////////////////////////////////////////////////////////////////////

void setup() { 
  size(1280,720);
  scbasedPhoto = new SimpleTweet(this); 
  scbasedPhoto.setOAuthConsumerKey("//////////"); 
  scbasedPhoto.setOAuthConsumerSecret("//////////"); 
  scbasedPhoto.setOAuthAccessToken("//////////"); 
  scbasedPhoto.setOAuthAccessTokenSecret("//////////"); 
  cols = width / videoScale; 
  rows = height / videoScale; 
  video = new Capture(this, cols, rows); 
  video.start(); 
  movie = new Movie(this, "BioCore.mp4"); 
  movie.loop(); 
  D = new Differentiation();
  
  ///////////////////////////////////////////////////////////////////////////////////////////////////////
    ConfigurationBuilder cb = new ConfigurationBuilder();
    cb.setOAuthConsumerKey("//////////");
    cb.setOAuthConsumerSecret("//////////");
    cb.setOAuthAccessToken("//////////");
    cb.setOAuthAccessTokenSecret("//////////");
    TwitterFactory tf = new TwitterFactory(cb.build());
    twitter = tf.getInstance();
    tweet=loadImage("tweetLogo.png");
    getNewTweets();
    currentTweet = int(random(10));
    thread("refreshTweets");
}

//////////////////////////////////////////////////////////////////////////////////////////////////////

void captureEvent(Capture video) { 
  video.read();
} 
void movieEvent(Movie movie) { 
  movie.read();
} 

/////////////////////////////////////////////////////////////////////////////////////////////////////////

void draw() { 
  background(0); 
  PVector cellInterSignals = new PVector(0.001, 0.0001); 
  PVector cellExterSignals = new PVector(0.0005, 0.001); 
  D.applyForce(cellInterSignals); 
  D.applyForce(cellExterSignals); 
  D.update(); 
  D.checkEdges(); 
  D.display(); 
  
  /////////////////////////////////////////////////////////////////////////////////////////////////////////////
    currentTweet = currentTweet + 1;
    if (currentTweet >= tweets.size()){
       currentTweet = int(random(10));
    }
    Status status = tweets.get(currentTweet);
    float x = 200;  //random(width);
    float y= 660; //random(height);
    int tweetsCount= currentTweet;
    noStroke();
    fill(0,150);
    rect(0, 655, 600, 115);
    image(tweet,x-155, y-25,45,37);
    stroke(255);
    line(x-128,y-45,x-128,y+40);
    line(x-128,y,x-180, y);
    fill(255);
    textSize(10);
    text(status.getText()+"by @", x-20, y, 200, 100);
    delay(0);
    textSize(20);
    text(tweetsCount, x-162, y+25);
} 

////////////////////////////////////////////////////////////////////////////////////////////////////////////

void getNewTweets(){
    try{
        Query query = new Query(searchString);
        QueryResult result = twitter.search(query);
        tweets = (ArrayList)result.getTweets();//tweets = result.getTweets();  
    }
    catch (TwitterException te){
        System.out.println("Failed to search tweets: " + te.getMessage());
        System.exit(-1);
    }
}

void refreshTweets(){
    while (true){
        getNewTweets();
        println("Updated Tweets");
        delay(1000);
    }
}

////////////////////////////////////////////////////////////////////////////////////////////////////////////

class Differentiation { 
  PVector bioAction; 
  PVector velocity; 
  PVector acceleration; 
  float mass; 
  Differentiation() { 
    bioAction = new PVector(0, 0); 
    velocity = new PVector(0, 0); 
    acceleration = new PVector(0, 0); 
    mass = currentTweet;
  } 

  //////////////////////////////////////////////////////////////////////////////////////////////////////////

  void applyForce(PVector force) { 
    PVector f = PVector.div(force, mass); 
    acceleration.add(f);
  } 

  //////////////////////////////////////////////////////////////////////////////////////////////////////////

  void update() { 
    //acceleration=PVector.random2D();
    velocity.add(acceleration); 
    bioAction.add(velocity); 
    acceleration.mult(0);
  } 

  /////////////////////////////////////////////////////////////////////////////////////////////////////////////

  void checkEdges() { 
    if (bioAction.x > width/2) { 
      bioAction.x = width/2; 
      velocity.x *= -1;
    } else if (bioAction.x < 0) { 
      bioAction.x = 0; 
      velocity.x *= -1;
    } 
    if (bioAction.y > height/2) { 
      bioAction.y = height/2; 
      velocity.y *= -1;
    } else if (bioAction.y<0) { 
      bioAction.y=0; 
      velocity.y*=-1;
    }
  } 

  /////////////////////////////////////////////////////////////////////////////////////////////////////////////

  void display() { 
    imageMode(CENTER); 
    image(movie, width/2, height/2, width*2, height*2); 
    video.loadPixels(); 
    movie.loadPixels(); 
    //movie.loadPixels(); 
    for (int i = 0; i < cols; i++) { 
      for (int j = 0; j < rows; j++) { 
        int x = i*videoScale; 
        int y = j*videoScale; 
        int loc1 = (video.width - i - 1) + j * video.width; 
        int loc2 = (movie.width - i - 1) + j * movie.width; 
        color c1 = video.pixels[loc1]; 
        float sz = (brightness(c1-int(bioAction.y))/70) * videoScale; 
        color b= movie.get(int(x), int(y)); 
        fill(random(255), random(loc2), random(b), bioAction.x); 
        strokeWeight(1); 
        stroke(0); 
        rectMode(CENTER); 
        rect(x + videoScale/2, y + videoScale/2, sz, sz);
      }
    }
  }
} 

/////////////////////////////////////////////////////////////////////////////////////////////////////////

void mousePressed() { 
  String tweet = scbasedPhoto.tweetImage(get(), "This Image Generated and Automatically Posted by BioPixels. It is a Stem Cells-Based Interactive-Generative Interface"); 
  println("Posted " + tweet);
} 
