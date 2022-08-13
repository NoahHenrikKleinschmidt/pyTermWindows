.. PyTermWindows documentation master file, created by
   sphinx-quickstart on Sat Aug 13 12:34:08 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to PyTermWindows's documentation!
=========================================

``PyTermWindows`` is a Python library for creating terminal windows.
It offers classes to display a terminal window filled with custom content and is a wrapper around python's built-in ``curses`` library.
The use of these classes is relatively straightforward and offers many handy features such as `to_next_line`, or `auto_scroll`
to make terminal window writing more readable and easier.


Simple Example
==============

The useage is relatively simple. 
In order to use `pytermwindows`, we create a new window class, let it inherit from either `Window` or `ScrollWindow`. 
Then we define a custom `contents` method to add contents to the window. Finally, we can set up a window instance in our code, 
and use the `run` method to activate the window. 

.. code-block:: python
      
   class MyWindow( Window ):
      """ A simple example to showcase the use of pytermwindows """

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


Once we are happy with our new window. We can make a new instance of it somewhere in our code using:

.. code-block:: python
   
   mywindow = MyWindow()
   mywindow.run()

`self.next_line` versus `self.to_next_line`
-------------------------------------------

There is an important difference between `self.next_line` and `self.to_next_line` (or any `self.{}_line` vs. `self.to_{}_line`).
The difference is that the `to_` method will automatically increment the current line position, while the `next_line` method will not.
Hence, if the window started at line 0, writing to the next line using the `self.next_line` method will write to line 1 but the current line will
still be 0. On the other hand, after calling `self.to_next_line` the current line will be 1.
The same prinicple applies to `self.to_first_line` or `self.to_previous_line`. 

The example below may help to further clarify this:

.. code-block:: python

   class LineExamples( Window ):
      def contents( self, **kwargs ):

         self.write( self.to_first_line, 0, f"({self.current_line}) Set the 'current_line' to the first allowed line in the window" )

         self.write( self.to_next_line, 0, f"({self.current_line}) Set the 'current_line' to the next line in the window after the first" )

         self.write( self.current_line, 80, f"<<< ({self.current_line}) Overwriting parts of the current line..." )

         self.write( self.previous_line, 80, f"< ({self.current_line}) make an insertion in the previous line but keep the 'current_line'  >" )

         self.write( self.next_line, 80, f"< ({self.current_line}) make an insertion in the next line but keep the 'current_line' >" )

         self.write( self.to_next_line, 0, f"{'-' * 20} ({self.current_line}) to the next line {'-' * 20}" )

         self.quit_on( keystring = "q" )
         self.refresh()

The resulting window looks like this, where the current line is always given in brackets:

.. code-block::

   (0) Set the 'current_line' to the first allowed line in the window              < (1) make an insertion in the previous line but keep the 'current_line'  >
   (1) Set the 'current_line' to the next line in the window after the first       <<< (1) Overwriting parts of the current line...
   -------------------- (2) to the next line --------------------                  < (1) make an insertion in the next line but keep the 'current_line' >


The "first allowed line" is one of the arguments accepted by the ``Window`` class. The default is 0.
However, we could specify the first line to be 4. In this case we can still write manually to lines 0,1,2, and 3, 
however, the window will only automatically start writing content from line 4. Like this we couls specify a header for our window
and then only use the automatic line tracking for writing dynamic content such as data.

.. code-block:: python

      
   class ShowMyData( Window ):
         def contents( self, **kwargs ):

            # write our header
            self.write( 0, 0, "This is my data" )
            self.write( 1, 0, "-" * 50 )

            data = kwargs.pop( "data", None )
            if data is None:
               self.write( self.to_first_line, 0, "No data was passed to the window" )
            else:
               self.to_first_line
               for line in data:
                  self.write( self.to_next_line, 0, line )

            self.quit_on( keystring = "q" )
            self.refresh()

We specify that the "first line" is supposed to be line 2 (3rd line). 
That means `self.first_line` will consider the index 2 as benchmark and ignore 0,1, but we can still manually write our headers there.

Which we can now call by:

.. code-block:: python
      
   w = ShowMyData( start_line = 2, height = 50, width = 50 )

   w.run( data = [ f"This is line {i+1}" for i in range(5) ] )
         

The resulting window looks like this:

.. code-block:: 

   This is my data
   --------------------------------------------------

   This is line 1
   This is line 2
   This is line 3
   This is line 4
   This is line 5


You may notice that there is a blank line between the header line and "This is line 1". This is because we 
go to the first line using `self.to_first_line` but we only start writing using `self.to_next_line`, which will automatically
increment to the next line before writing.

We need to first call `self.to_first_line`, however, so that we can reset the window lines after each refresh. If we did not have the
`self.to_first_line` then after every refresh we would keep writing our lines time after time until we ran out of lines to write to.
This would look like this:

.. code-block:: 

   This is my data
   --------------------------------------------------

   This is line 1
   This is line 2
   This is line 3
   This is line 4
   This is line 5
   This is line 1
   This is line 2
   This is line 3
   This is line 4
   This is line 5
   This is line 1
   This is line 2
   This is line 3
   This is line 4
   This is line 5

In case the blank line bothers us, we could either specify the first line as `1` instead of `2` and thereby make sure we actually start at 2 since we only write using `self.to_next_line`, 
or we change the `contents method` to write the first data entry using `self.to_first_line` and all others using `self.to_next_line`.

..  code-block:: python

   # this would remove the blank line between the header and our data
   w = ShowMyData( start_line = 1, height = 50, width = 50 )
   w.run( data = [ f"This is line {i+1}" for i in range(5) ] )

   # or this would also work (within the contents method)
   self.write( self.to_first_line, 0, data[0] )
   for line in data[1:]:
      self.write( self.to_next_line, 0, line )

.. code-block:: 

   This is my data
   --------------------------------------------------
   This is line 1
   This is line 2
   This is line 3
   This is line 4
   This is line 5







.. pytermwindows.Window class
.. --------------------------

.. .. automodule:: pytermwindows.Window
..    :members:
..    :undoc-members:
..    :show-inheritance:

.. pytermwindows.ScrollWindow class
.. --------------------------------

.. .. automodule:: pytermwindows.ScrollWindow
..    :members:
..    :undoc-members:
..    :show-inheritance:



.. toctree::
   :maxdepth: 2
   :caption: Contents:

   Window
   ScrollWindow

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
