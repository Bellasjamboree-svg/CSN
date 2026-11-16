# CSN
import oscP5.*; 
import netP5.*; 

int numtracks = 35; 
wigBtn[] buttons = new wigBtn[numtracks]; 
wigVis vis; 
float[] vuL = new float[100]; //allow up to 100 tracks 
float[] vuR = new float[100]; //allow up to 100 tracks 
//oscP5 library is a class.  Make a variable of this type   
OscP5 oscP5; 

//netP5 provides a NetAddress class enabling us to send data over networks 
NetAddress Reaper; 

void setup() { 
  size(900,400); 
  frameRate(25); 
  /* create an instance of oscP5 */ 
  /* and start listening for incoming messages at port 9000 */ 
  oscP5 = new OscP5(this,9000); 
  /* Reaper is a NetAddress. a NetAddress takes 2 parameters, 
   * an ip address and a port number. Reaper is used as parameter in 
   * oscP5.send() when sending osc packets to Reaper which is on the same 
   * computer (127.0.0.1) and listening on port 8000. 
   */ 
   Reaper = new NetAddress("127.0.0.1",8000); 
   for(int i=0;i<buttons.length;i++) 
     buttons[i] = new wigBtn(10+i*25,20,20); 
  sendOSC("/device/track/count",numtracks);      
  vis = new wigVis(numtracks); 
} 

void draw() { 
  background(0); 
  for(int i=0;i<buttons.length;i++) 
  { 

    buttons[i].offc = int(vuL[i+1]*255); 
    buttons[i].draw(); 
  } 

  vis.draw(vuL,200,200,150,color(255),color(255,0,0));  
  vis.draw(vuR,200,200,100,color(255),color(0,0,255,128)); 
} 
void mousePressed() { 
  for(int i=0;i<buttons.length;i++) 
  { 
    if(buttons[i].togglebtn()) 
    { 
      //if togglebutn function returns true, it was pressed, so send OSC message 
      //use i+1 as 0 is master, don't need to solo that really! 
      OscMessage myMessage = new OscMessage("/track/"+(i+1)+"/solo"); 
      if(buttons[i].btnon) 
        myMessage.add(1.0); /* add a float to the osc message */  
      else  
        myMessage.add(0.0);     
      oscP5.send(myMessage, Reaper);  
    } 
  } 
} 


//when an OSC message is received, the oscEvent function is called 
//much the same as a mousePressed function etc.  The OSC message 
//is passed as data of type OscMessage 
void oscEvent(OscMessage theOscMessage) { 
  String addr = theOscMessage.addrPattern(); 
  String[] adarray = split(addr,"/"); 
  int trnum=0; 

  if(adarray.length>3 && adarray[1].equals("track") && adarray[3].equals("solo")) 
  { 
    trnum = int(adarray[2]); 
    //trnum-1 as tracks start at 1, but array starts at 0 
    //also, it's impossible to convert from a float to a boolean, so have to convert 
    //to an int first! 
    buttons[trnum-1].confirmedstate = boolean(int(theOscMessage.get(0).floatValue()));   
  } 

  else if(adarray.length>4 && adarray[1].equals("track") && adarray[3].equals("vu")) 
  { 
    trnum = int(adarray[2]); 
    if(adarray[4].equals("L")) 
      vuL[trnum] = theOscMessage.get(0).floatValue(); 
    else 
      vuR[trnum] = theOscMessage.get(0).floatValue(); 
  }   
  //catch MASTER VU data 
  else if(adarray.length>1 && adarray[1].equals("master")) 
  { 
    if(adarray.length>2 && adarray[2].equals("vu")) 
    { 
      if(adarray.length>3 && adarray[3].equals("L")) 
        vuL[0] = theOscMessage.get(0).floatValue(); 
      else if(adarray.length>3 && adarray[3].equals("R")) 
        vuR[0] = theOscMessage.get(0).floatValue(); 
    }  
  } 
  else 
  { 
    /* print unhandled address pattern and the typetag of the received OscMessage */ 

    print("### received an osc message."); 
    print(" addrpattern: "+theOscMessage.addrPattern()); 
    println(" typetag: "+theOscMessage.typetag()); 
  }
} 

void sendOSC(String s,float f) 
{   
    OscMessage myMessage = new OscMessage(s);     
    myMessage.add(f);   
    oscP5.send(myMessage,Reaper); 
} 

void sendOSC(String s,int i) 
{   

    OscMessage myMessage = new OscMessage(s);     
    myMessage.add(i);   
    oscP5.send(myMessage,Reaper); 
} 

void sendOSC(String s) 
{   

    OscMessage myMessage = new OscMessage(s);     
    oscP5.send(myMessage,Reaper); 
} 
