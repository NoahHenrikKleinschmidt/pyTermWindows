.. _ScrollWindow:


The ScrollWindow
================

The `ScrollWindow` class is a daugther of the `Window` class. It is used to create a window that can be scrolled up and down.
To re-use our `ShowMyData` class from before, we could use the `ScrollWindow` class to display only parts of our lines.
For instance, we could decide only to write 10 lines of our data at a time:

.. code-block:: python

      
   class MyScrollData( ScrollWindow ):
         def contents( self, **kwargs ):

            # write our header
            self.write( 0, 0, "This is my data" )
            self.write( 1, 0, "-" * 50 )

            data = kwargs.get( "data", None )
            
            if data is None:
               self.write( self.to_first_line, 0, "No data was passed to the window" )

            else:

               # we subset to the lines within our scrolling range
               subset = data[ self.scroll_range( as_slice = True ) ]
               # there is also a pre-defined method to do this:
               # subset = self.crop_data_to_scroll_range( data )

               # then we write our data
               self.to_first_line
               for line in subset:
                  self.write( self.to_next_line, 0, line )


               self.write( self.to_next_line, 0, f"{'-' * 10} end of scroll-range {'-' * 10}" )

            # now we enable the scrolling, and restrict the scrolling to make sure that the window dimensios do not interfer with our scrolling.
            # This is useful when our window is much larger or smaller than our data. In this case we might either scroll past our data 
            # or not be able to scroll all our data, because by default the window height is used as scrolling reference.
            self.auto_scroll( restrict = len(data) )

            self.quit_on( keystring = "q" )
            self.refresh()

We can now call this and get a scrollable window 

.. code-block:: python

   w = MyScrollData( start_line = 1, height = 50, width = 50 )

   # now we set the scroll range to 10 entries 
   w.set_scroll_range( 10 )

   w.run( data = [ f"This is line {i+1}" for i in range(100) ] )


And our final window looks like this (scrolled a little bit):

.. code-block:: 

   This is my data
   --------------------------------------------------
   This is line 5
   This is line 6
   This is line 7
   This is line 8
   This is line 9
   This is line 10
   This is line 11
   This is line 12
   This is line 13
   This is line 14
   ---------- end of scroll-range ----------

pytermwindows.ScrollWindow class
--------------------------

.. automodule:: pytermwindows.ScrollWindow
   :members:
   :undoc-members:
   :show-inheritance: