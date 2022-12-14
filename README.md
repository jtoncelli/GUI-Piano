# GUI-Piano
A GUI project using Javafx to create a playable piano
----------

This project was my first big GUI project, and I spent most of my time with it experimenting with various features of **javafx**. The final product is a playable piano with some extra features, including:
- **"clickable" keys**, i.e; mouse inputs
- a **keyboard interface**, i.e; keyboard input for the notes
- a choice of **several octaves** totaling to 88 keys
- **different instrument** selection
- **pre-written** and playable songs

For further information on the logistics of the project, please read below: 


-------------

### To **create the canvas**:

Piano keys obviously have a very specific and recognisable look, and I wanted to maintain that look for the best user experience. To do this, I used a digital image of piano keys, one similar to [**this**](https://upload.wikimedia.org/wikipedia/commons/thumb/1/15/PianoKeyboard.svg/161px-PianoKeyboard.svg.png?20061008130835), and measured the relative sizes of the height and width of the white and black keys. 

**The units used are irrelevant**, whether its pixels, inches, centimeters, or anything else, so long as the same unit is consistently used. With these measurements, I recorded **the ratios of the keys in relation to each other**, as well as some other useful numbers, as shown here. 

```java
    /*
        white key ratio width : height -- 1 : 5
        black key ratio width : height -- 1.5 : 7 ( for white height of 10 )
        if height = 10, black key starts 4 from bottom
        one octave is 13 keys, 5 black, 8 white ( from low C to high C )
        ratio of canvas size width : height -- 8 : 5
    */

```

To actually draw the keys onto the canvas, I made a helper function, ***drawShapes()***. Instead of creating each key as a button, this method accepted the height and width of the canvas and used some light geometry to split it up according to the defined ratios of the keys. For white keys, I utilized the ***strokeLine*** method to draw their seperating lines. For black keys, I used the ***fillRect()*** method built into javafx canvas. 

```java
    private void drawShapes(GraphicsContext gc, int height, int width) {
        /*
        draws one octave of keys based on geometry
        splits up the screen equally for each key, and uses the ratios defined above
        --------------------------------------------
        fillRect is used for black keys, and accepts two coordinates 
        the first coordinate is the top left corner, and the second coordinate is the bottom right corner
        
        gc.fillRect(int x1, int y1, int x2, int y2)
        --------------------------------------------
        strokeLine is used for drawing the lines that seperate white keys
        it accepts two coordinates for the start and end point of the line
        
        gc.strokeLine(int x1, int y1, int x2, int y2);
        */
    }

```

-------------

### To play the notes:

As part of initialization, two arrays are created, **noteThreads** and **notesOn**. 

```java
    private Thread[] noteThreads = new Thread[14];
    private boolean[] notesOn = new boolean[14];
```

noteThreads is used to allocate a individual thread to play for each note. Doing so allows for multiple notes to be played simultaneously. notesOn is an array of booleans that represents whether a note is currently playing or not. notesOn is used as a layer of protection against trying to play a note on an already active thread.

----------------

By making a visual display instead of a series of buttons, I had to create my own way of collecting mouse inputs. Luckily, the MouseEvent class has the built in classes of ***getX()*** and ***getY()***, which return the x and y coordinates, respectively, of where the cursor has been clicked. Based on those values, the helper method ***getNote()*** uses a series of **if** statements to determine which key the click is on. 

When the mouse is clicked, an EventHandler calls getNote(), and passes the string into a seperate helper method, ***playThreadedNote()***, and this method will activate the correct thread within noteThreads. When the thread is activated, the computer on which the program is being run should play the note(if it has functioning speakers). 

```java
    canvas.setOnMousePressed(new EventHandler<MouseEvent>() {
        @Override
        public void handle(MouseEvent event) {
            try {
                 playThreadedNote(getNote(event, width, height), (int) octaveSlider.getValue(), noteThreads);
            }catch(Exception e){}
        }
    });
```


In order to stop playing the note, the thread must be turned off. A seperate EventHandler from the one used to activate the thread is called when the mouse is released. The getNote() method is used to determine what key the cursor is over, and the thread within noteThreads is interrupted.

```java
    canvas.setOnMouseReleased(new EventHandler<MouseEvent>() {
        @Override
        public void handle(MouseEvent event) {
           noteThreads[getNoteNum(getNote(event, width, height))].interrupt();
        }});
```


------

### To play a song:

I wanted to build in songs that would play at the press of a button, but there aren't any built in ways to do that other than to play each note of the song in succession. I decided to make the helper classes, Song and Note, which would act as the "sheet music" for the piano. 

The [**Note**](https://github.com/jtoncelli/GUI-Piano/blob/965f2b28909a51239bdc18e9a3c812826dd50694/src/java/Note.java) class has two attributes, a String representation of the note and a double for its duration. The duration is a decimal between 0 and 1, which represents the percentage of a measure in 4/4 timing, so .25 is a quarter note, .5 is a half note, 1 is a whole note, etc. 

```java
    public class Note{
        private String note;
        private double duration;
    }
```

---------

The [**Song**](https://github.com/jtoncelli/GUI-Piano/blob/965f2b28909a51239bdc18e9a3c812826dd50694/src/java/Song.java) class only has three attributes, a String name and an integer bpm (beats per minute). The bpm is used to convert the timing of notes into milliseconds. The most important class attribute, however, is the ArrayList of Notes, **sheetMusic**. sheetMusic contains all the notes and their respective durations. 

```java
    public class Song{
        private String name;
        private int bpm;
        private ArrayList<Note> sheetMusic = new ArrayList<Note>();
    }
```
---------

In order to fill sheetMusic in a better way than simply apoending all the notes to the ArrayList manually, I made up a String representation of a Note object, which has three main components. First, a number to indicate the octave of the note, ranging from 0-7. Next is the string of the note itself, such as "C" or "D#". The final component is a decimal immediately after the note which represents its duration in a 4/4 measure. An example of a properly formatted String to convert to a Note is "6C.25", or "5A.5". 

Some examples of properly formatted notes within a txt file:
```txt
    6C.25 6C.25
    6G.25 6G.25
    6A.25 6A.25
```

You can look at the whole file [**here**](https://github.com/jtoncelli/GUI-Piano/blob/965f2b28909a51239bdc18e9a3c812826dd50694/src/java/twinkleSong.txt)

---------

Input text is accepted in a method in the Song class, parseNotes(), which takes either a String or a file. The text is then processed as described above and the correctly formatted notes are added to sheetMusic, in order. Looking back, I could have made a constructor for the Note class that parses a string, but it works just as well within the Song class. 

---------

Finally, within the GUI_Synth class, I made a drop down menu where the user can choose which song to play and an associated button to play it. Pushing the button calls on the **playSheetMusic()** method, which will play through each note in the sheetMusic ArrayList. For each note, it will convert the song's bpm into milliseconds and play the associated 

I would also like to add something to pause or interrupt the song, because as of now it can't be interrupted by the user. 
