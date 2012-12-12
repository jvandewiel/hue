// Control Philips Hue light using audio input that dos beat detection
// You need to have the HttpClient and Minim libraries installed
// HttpClient must be in a folder named "core" where the skecth resides (e.g. ../skethfolder/core)
// Minim is installed with the newest version of Processing

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.DefaultHttpClient;
import java.io.*;
import ddf.minim.*;
import ddf.minim.effects.*;
import ddf.minim.analysis.*;

// Global variables
Minim minim;
AudioInput in; // audio stream from microphone/line in
// beat detection
BeatDetect beat;
BeatListener bl;
// Hue objects
HueHub hub; // the hub
HueLight l_one; // light instances
HueLight l_two;
HueLight l_three;

// FFT info 
FFT fft; 
BandPass bpf;

// timing and fps
int fps = 20; // framerate, used to translate draw() to real world seconds
int flashTime = 200; // duration of a flash in ms

// init stuff
void setup()
{
  size(512, 400, P3D);
  frameRate(fps);  // Frames per second

  // init minim and beat detection
  minim = new Minim(this);
  in = minim.getLineIn(minim.MONO); // could use stereo, but for this purpose mono is fine
  // see class below for more info
  
  beat = new BeatDetect();
  beat.detectMode(BeatDetect.FREQ_ENERGY); // want to do freq analysis, not energy 
  beat.setSensitivity(50); // ms to wait before next analysis, range 10 .. max int 
  bl = new BeatListener(beat, in);

  // init for graphics
  ellipseMode(CENTER);
  // set colormode, hue max value 65000, sat/bri = 255 so screen ~ light  
  colorMode(HSB, 65000, 255, 255);
  rectMode(CORNERS);

  // init for hub/lights
  hub = new HueHub();  
  l_one = new HueLight(1, hub);
  l_two = new HueLight(2, hub);
  l_three = new HueLight(3, hub);

  // set position on screen
  l_one.setPosition(100, 100);
  l_two.setPosition(250, 100);
  l_three.setPosition(400, 100);

  /*
    some more fft init
   bpf filter helps beatDetection algo when bass beat detection is only wanted  
   freq - the center frequency of the band to pass (in Hz)
   bandWidth - the width of the band to pass (in Hz)
   sampleRate - the sample rate of audio that will be filtered by this filter
   */
   //http://code.compartmental.net/tools/minim/manual-iirfilter/
  bpf = new BandPass(100, 100, in.sampleRate());
  in.addEffect(bpf);
  
  fft = new FFT(in.bufferSize(), in.sampleRate());
  // calculate averages based on a miminum octave width of 22 Hz
  // split each octave into n bands
  fft.logAverages(22, 3); // n = 1
}

void draw() {
  background(0);

  //paint lights (always; logic handled in draw())
  l_one.draw();
  l_two.draw();
  l_three.draw();

  // triggers l_one, only if the light is not yet on
  if ( beat.isKick() && !l_one.isOn()) {
    l_one.incHue(); // increase hue color
    l_one.flashLight(flashTime);
  }

  // triggers 2
  if ( beat.isHat() && !l_two.isOn() ) {
    l_two.incHue();
    l_two.flashLight(flashTime);
  }

  // triggers 3
  if ( beat.isSnare() && !l_three.isOn() ) {
    l_three.incHue();
    l_three.flashLight(flashTime);
  }
  /*
  // beat - alternative method using fft
   // println(fft.getAvg(2));
   if (fft.getAvg(2) > 15 && !l_three.isOn()) {
   println("beat");
   // l_three.incHue();
   // l_three.flashLight(flashTime);
   }
   */

  // draw audio wave
  stroke(10000, 255, 255);
  // MONO input, so only use left buffer
  for (int i = 0; i < in.bufferSize() - 1; i++) {
    float left1 = 50 + in.left.get(i) * 50;
    float left2 = 50 + in.left.get(i+1) * 50;
    line(i, left1, i+1, left2);
    // if stereo, lines to draw right channel
    /*
    float right1 = 60 + in.right.get(i) * 50;
     float right2 = 60 + in.right.get(i+1) * 50;
     line(i, right1, i+1, right2);
     */
  }

  // do FFT, draw spectogram
  fft.forward(in.mix);
  stroke(10000, 255, 255);
  fill(20000, 255, 255);
  // width of the spec basrs
  int w = int(width/fft.avgSize());

  // draw full spectrum
  /*
  for (int i = 0; i < fft.specSize(); i++) {
   line(i, height/2, i, height/2 - fft.getBand(i)*2);
   } 
   */

  // draw average spectrum values
  for (int i = 0; i < fft.avgSize(); i++) {
    //println(fft.getAvg(i));
    rect(i*w, height, i*w + w, height - fft.getAvg(i));
  }
}

// always close Minim audio classes when you are finished with them
void stop() {
  in.close();
  minim.stop();
  // close connection to hub
  hub.disconnect();
  // this closes the sketch
  super.stop();
}

// Beatlistener class
// see http://code.compartmental.net/tools/minim/manual-beatdetect/
class BeatListener implements AudioListener {
  private BeatDetect beat;
  private AudioInput source;

  BeatListener(BeatDetect beat, AudioInput source) {
    this.source = source;
    this.source.addListener(this);
    this.beat = beat;
  }

  void samples(float[] samps) {
    beat.detect(source.mix);
  }

  void samples(float[] sampsL, float[] sampsR) {
    beat.detect(source.mix);
  }
}

// Hub class, contains the http stuff as well
// Handles url as url = http://<IP>/api/<KEY>/lights/<id>/state";

class HueHub {
  private static final String KEY = "your key goes here"; // "secret" key/hash
  private static final String IP = "192.168.2.5"; // ip address of the hub
  private static final boolean ONLINE = true; // for debugging purposes, set to true to allow communication

  private DefaultHttpClient httpClient; // http client to send/receive data

  // constructor, init http
  public HueHub() {
    httpClient = new DefaultHttpClient();
  }

  // Query the hub for the name of a light
  public String getLightName(HueLight light) {
    // build string to get the name,   
    return "noname";
  }

  // apply the state for the passed hue light based on the values
  public void applyState(HueLight light) { 
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    try {
      // format url for specific light
      StringBuilder url = new StringBuilder("http://");
      url.append(IP);
      url.append("/api/");
      url.append(KEY);
      url.append("/lights/");
      url.append(light.getID());
      url.append("/state");
      // get the data from the light instance
      String data = light.getData();
      StringEntity se = new StringEntity(data);
      HttpPut httpPut = new HttpPut(url.toString());
      // debugging
      // println(url);
      //println(light.getID() + "->" + data);

      //with post requests you can use setParameters, however this is
      //the only way the put request works with the JSON parameters
      httpPut.setEntity(se);
      // println( "executing request: " + httpPut.getRequestLine() );

      // sending data to url is only executed when ONLINE = true
      if (ONLINE) { 
        HttpResponse response = httpClient.execute(httpPut);
        HttpEntity entity = response.getEntity();
        /*
        if (entity != null) {
          // only check for failures, eg [{"success":
          entity.writeTo(baos);
          //success = baos.toString();
          if (!baos.toString().startsWith("[{\"success\":")) println("error udating"); 
          println(baos.toString());
        }
        */
        // needs to be done to ensure next put can be executed and connection is released
        if (entity != null) entity.consumeContent();
      }
    } 
    catch( Exception e ) { 
      e.printStackTrace();
    }
  }

  // close connections and cleanup
  public void disconnect() {
    // when HttpClient instance is no longer needed, 
    // shut down the connection manager to ensure
    // deallocation of all system resources
    httpClient.getConnectionManager().shutdown();
  }
}

// Hue class; one instance represents a lamp which is addressed using number
class HueLight {
  private int id; // lamp number/ID as known by the hub, e.g. 1,2,3
  // light variables
  private int hue = 30000; // hue value for the lamp
  private int saturation = 255; // saturation value
  private int  brightness = 255; // brightness
  private boolean lightOn = false; // is the lamp on or off, true if on?
  private byte transitiontime = 0; // transition time, how fast  state change is executed -> 1 corresponds to 0.1s
  // hub variables
  private HueHub hub; // hub to register at
  private String name = "noname"; // set when registering the lamp with the hub
  // graphic settings
  private byte radius = 80; // size of the ellipse drawn on screen
  private int x; // x position on screen
  private int y; // y position on screen
  // control variables
  private float damping = 0.9; // control how fast dim() impacts brightness and lights turn off
  private float flashDuration = 0.2; // in approx. seconds
  private float frameCounter; // keeps track of number of frames drawn and turns light off when ==0

  // constructor, requires light ID and hub
  public HueLight(int lightID, HueHub aHub) {
    id = lightID;
    hub = aHub;
    // check if registered, get name [not implemented]
    name = hub.getLightName(this);
  }

  // cycle thrue the colors
  public void incHue() {
    hue += 1000;
    setHue(hue);
  }

  // set the hue value; if outside bounds set to min/max allowed
  public void setHue(int hueValue) {
    hue = hueValue;
    if (hue < 0 || hue > 65532) {
      hue = 0;
    }
  }

  // set the brightness value, max 255
  public void setBrightness(byte bri) {
    brightness = bri;
  }

  // set the saturation value, max 255
  public void setSaturation(byte sat) {
    saturation = sat;
  }

  // set the tranistion time; 1 = 0.1sec (not sure if there is a max)
  public void setTransitiontime(byte transTime) {
    transitiontime = transTime;
  }

  // returns true if the light is on (based on last state change, not a query of the light) 
  public boolean isOn() {
    return this.lightOn;
  }

  /*
   have the changes to the settings applied to the lamp & visualize; this
   calls the hub which handles the actual updates of the lights
   */
  public void update() {
    hub.applyState(this);
    // debugging
    // println("send update " + id);
  }

  // convenience method to turn the light off
  public void turnOff() {
    this.lightOn = false;
    update();
  }

  /* 
   turn on the light for <duration> ms; compensates for transition time 
   if duration = 2000 ms and fps = 20 -> #frames = (2000 / 1000) / (1/20) = 40 frames
   transition is subtracted before framecount is calculated
   */
  public void flashLight(float duration) {
    float durationOn = duration - (transitiontime * 100); // transition time of 1 = 0.1s = 100ms
    // translate the duration in ms to number of frames
    frameCounter = (duration/1000) / (1/frameRate);
    lightOn = true;
    // println("on for " + duration + "[ms], translates to " + frameCounter + "[frames] with fps " + frameRate);
    update();
  }

  // convenience method to turn the light on
  public void turnOn() {
    this.lightOn = true;
    update(); // apply new state
  }

  // convenience method to turn the light on with some passed settings
  public void turnOn(int hue, int brightness) {
    this.lightOn = true;
    this.hue = hue;
    this.brightness = brightness;
    update(); // apply new state
  }

  /* 
   return data with lamp settings, JSON formatted string, to be send to hub
   sometimes after a while you get an error message that the light is off
   and it won’t change, even when it’s actually on. You can work around 
   this by always turning the light on first. 
   */
  public String getData() {
    StringBuilder data = new StringBuilder("{");
    data.append("\"on\" :"); 
    data.append(lightOn);
    // only if the light is on we need the rest
    if (lightOn) {
      data.append(", \"hue\":");
      data.append(hue);
      data.append(", \"bri\":");
      data.append(brightness);
      data.append(", \"sat\":");
      data.append(saturation);
    }
    // always send transition time, to control how fast the state is changed
    data.append(", \"transitiontime\":");
    data.append(transitiontime);
    data.append("}");
    return data.toString();
  }

  // get current values
  public int getBrightness() {
    return brightness;
  }

  public int getSaturation() {
    return saturation;
  }

  public int getHue() {
    return hue;
  }

  public int getID() {
    return id;
  }

  /*
   dim the light using damping factor; if brightness < x and lighton then turn if
   this could allow for smoother on/off changes but also risk for hub errors (to any changes) 
   */
  public void dim() {
    brightness *= damping;
    if (brightness < 20 && lightOn) { // 20 is arbitrary threshold, could be higher/lower
      brightness = 0;
      lightOn=false;
      update();
    }
  }
  
  // set position on screen
  public void setPosition(int x, int y) {
    this.x = x;
    this.y = y;
  }

  // draw the light on screen 
  // always draw the light & text, only fill when lightOn and only call update when turning off
  public void draw() {
    // println("val fc:"  +frameCounter);
    // do we need to turn it off?
    if (frameCounter < 0 && lightOn) {
      turnOff();
    } 
    if (lightOn) {
      // if it is on, draw ellipse with fill and reduce #frames
      fill(hue, saturation, brightness);
      frameCounter -= 1;
    } 
    else {
      fill(0, 0, 0);
    }
    // draw ellipses
    ellipse(x, y, radius, radius);
    // add txt string to indicate lightname/id
    textSize(16);
    fill(65332, 255, 255);
    text(id, x-5, y+6); // approx. center
  }
}

