.. _Window:

The default Window
==================

We can make a simple window to show the contents of some iterable data input.

.. code-block:: python

   class ShowMyData( Window ):
         def contents( self, **kwargs ):

            # write our header
            self.write( self.to_first_line, 0, "This is my data" )
            self.write( self.to_next_line, 0, "-" * 50 )

            # get the data we want from the kwargs
            data = kwargs.pop( "data", None )
            if data is None:
               self.write( self.to_next_line, 0, "No data was passed to the window" )
            else:
               for line in data:
                  self.write( self.to_next_line, 0, line )

            # make sure that the window can be exited again
            self.quit_on( keystring = "q" )

            # and refresh the window contents
            self.refresh()

Now we can call this window within our code like this:

.. code-block:: python

   w = ShowMyData( height = 20, width = 50 ) # we give the window large enough dimensions
   mydata = [ f"This is my data ({i+1})" for i in range(10) ]
   w.run( data = mydata )

And the resulting window will look like this:

.. code-block:: 

    This is my data
    --------------------------------------------------
    This is my data (1)
    This is my data (2)
    This is my data (3)
    This is my data (4)
    This is my data (5)
    This is my data (6)
    This is my data (7)
    This is my data (8)
    This is my data (9)
    This is my data (10)

pytermwindows.Window class
--------------------------

.. automodule:: pytermwindows.Window
   :members:
   :undoc-members:
   :show-inheritance: