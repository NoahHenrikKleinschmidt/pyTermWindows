# PyTermWindows

This is `PyTermWindows` a `curses`-based package defining the basic `Window` and `ScrollWindow` classes to display data through a terminal window. The classes are proxies for an `curses.stdscr` (called _window_) and allow easy setup, and access to key properties. Also, through atrributes such as `next_line` or `first_line` they allow readable and dynamic content addition. 

The code of the `ScrollWindow` class was especially inspired by [mingrammer's great scroll window example](https://github.com/mingrammer/python-curses-scroll-example). If you are interested in learning more about scrolling terminal windows, check it out!

Examples
--------

The useage is relatively simple. We create a new window class, let it inherit from either `Window` or `ScrollWindow` and define a custom `contents` method to add contents to the window. Then, we can set up a window instance in our code, and use the `run` method to activate the window. 

```python
class MyWindow( Window ):

    def contents( self, **kwargs ):
        """
        Here will be our own contents.

        Note
        ----
        The kwargs are passed down from the `run` method to the `contents` method...

        """
        # write a header
        self.write( 
                    # the line to write to
                    self.to_first_line, 
                    # the position within the line to start writing
                    0,
                    # the text to display
                    "This is my great window"
                )
        
        # and an underline
        self.write( self.to_next_line, 0, "-" * 50 )

        # add some responsiveness to the window

        # get the keypress event as a string (we could use self.keycode to get the ord code instead)
        # both keystring and keycode will be None in case no keys were pressed.
        key = self.keystring

        if key is not None:
            self.write( self.to_next_line, 0, f"You pressed '{key}'..." )
            
        # now let the window quit if we press "q"
        # we could either use the string or the numeric (ord) code
        self.quit_on( keystring = "q" )

        # finally, we need to refresh
        self.refresh()


# and now run our window
mywindow = MyWindow( height = 10, width = 50 )
mywindow.run()
```

The resulting window looks like this:

```
This is my great window
--------------------------------------------------
You pressed 'g'...

```

A more complex example of a ScrollWindow: Let's say we have a data file we wish to read from every few seconds and display the lines in a ScrollWindow. To make that easier, the `Window` class (and by inheritance also the ScrollWindow) offers a `set_update_interval` method that will control an external timer to allow method calling outside of the window-frame updating timescale.

```python

class MyScroller( ScrollWindow ):
        """
        This class shall read my log file and display the lines.

        Parameters
        ----------
        
        filename : str
            The file to read.
        
        **kwargs
            Any keyword arguments to be passed to the ScrollWindow constructor.
        """
        def __init__( self, filename, **kwargs ):
            super().__init__( **kwargs )
            self.file = filename
            self.file_contents = None # and setup the attribute to store the contents of the log file.
        
        def read( self ):
            """
            Read the given file.
            
            Returns
            -------
            list
                A list of lines in the file.
            """
            with open( self.file, "r" ) as f:
                return f.readlines() 
        
        def contents( self, **kwargs ):
            """
            The contents of our Logfile
            """

            # write a nice header again
            self.write( self.to_first_line, 0, f"File: {self.file}" )
            self.write( self.to_next_line, 0, "-" * 50 )

            # now read the file the first time (None), or whenever 
            # our update timer says it's okey to read again.
            if self.can_update() or self.file_contents is None : 
                self.file_contents = self.read()
                
            # now display our data. We need to crop the data to the scroll range in order to only display
            # a subset of the data 
            data = self.file_contents[ self.scroll_range( as_slice = True ) ]
            for line in data:
                self.write( self.to_next_line, 0, line )

            # now we need to enable the scrolling functionality.
            # ScrollWindow offers the handy method auto_scroll that will adjust the 
            # scroll range for us. Using the restrict argument we make sure that we 
            # keep all of our data in the scrolling window and not scroll out of window...
            self.auto_scroll( restrict = len(self.file_contents) ) 

            # now we make sure we can exit again
            self.quit_on( keystring = "q" )

            # AND we MUST NOT FORGET increment the UPDATE COUNTER for file reading!
            self.update_counter()

            # finally we can refresh the window and we're done...
            self.refresh()


# now we can use our window as:
myscroller = MyScroller( "my_file.txt", height = 20, width = 100 )

# setup the update interval as (approx) every 10 seconds
# hence, our log file contents will be updated every 10 seconds
myscroller.set_update_interval( 10 )

# and set the scroll range (i.e. how many data lines to display)
# let's say 5 lines at a time.
myscroller.set_scroll_range( 5 )

myscroller.run()
```

The resulting window looks like this:

```
File: ~/GIT/pyTermWindows/my_file.txt
--------------------------------------------------
8888757      pall smk-roh- lzeitler PD       0:00      1 (AssocMaxJobsLimit)
8888758      pall smk-roh- lzeitler PD       0:00      1 (AssocMaxJobsLimit)
8888734      pall smk-sim- lzeitler PD       0:00      1 (AssocMaxJobsLimit)
8888735      pall smk-sim- lzeitler PD       0:00      1 (AssocMaxJobsLimit)
8888736      pall smk-sim- lzeitler PD       0:00      1 (AssocMaxJobsLimit)
```