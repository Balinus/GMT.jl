GMT.jl
======

Julia wrapper for the Generic Mapping Tools

There is no manual yet but looking into the tests.jl should give a (good) idea on how it works. Note also that this wrapper works only with the GMT5.2 developing version.

Using
=====

The Julia wrapper was designed to work in a way the closest as possible to the command line version and yet to provide all the facilities of the Julia language. In this sense, all **GMT** options are put in a single text string that is passed, plus the data itself when it applies, to the ``gmt()`` command. For example to reproduce the CookBook example of an Hemisphere map using a Azimuthal projection

>gmt("pscoast -Rg -JA280/30/3.5i -Bg -Dc -A1000 -Gnavy -P > GMT_lambert_az_hemi.ps")

but that is not particularly interesting as after all we could do the exact same thing on the a shell command line. Things start to get interesting when we can send data *in* and *out* from Julia to
**GMT**. So, consider the following example

    t = rand(100,3) * 150;
    G = gmt("surface -R0/150/0/150 -I1", t);

Here we just created a random data *100x3* matrix and told **GMT** to grid it using it's program *surface*. Note how the syntax follows closely the standard usage but we sent the data to be interpolated (the *t* matrix) as the second argument to the ``gmt()`` function. And on return we got the *G* variable that is a type holding the grid and it's metadata. See the :ref:`grid struct <grid-struct>` for the details of its members.

Imagining that we want to plot that random data art, we can do it with a call to *grdimage*, like

    gmt("grdimage -JX8c -Ba -P -Cblue,red > crap_img.ps", G)

Note that we now sent the *G grid* as argument instead of the **-G** *gridname* that we would have used in the command line. But for readability we could well had left the **-G** option in command string. E.g:

    gmt("grdimage -JX8c -Ba -P -Cblue,red -G > crap_img.ps", G)

While for this particular case it makes no difference to use or not the **-G**, because there is **only** one input, the same does not hold true when we have more than one. For example, we can run the same example but compute the color palette separately.

    cpt = gmt("grd2cpt -Cblue,red", G);
    gmt("grdimage -JX8c -Ba -P -C -G > crap_img.ps", cpt, G)

Now we had to explicitly write the **-C** & **-G** (well, actually we could have omitted the **-G** because it's a mandatory input but that would make the things more confusing). Note also the order of the input data variables. It is crucial that they are used in the **exact** same order as the options in the command string.

To illustrate another aspect on the importance of the order of input data let us see how to plot a sinus curve made of colored filled circles.

    x = linspace(-pi, pi)';            # The xx var
    seno = sin(x);                     # yy
    xyz  = [x seno seno];              # Duplicate yy so that it can be colored
    cpt  = gmt("makecpt -T-1/1/0.1");  # Create a color palette
    gmt("psxy -R-3.2/3.2/-1.1/1.1 -JX12c -Sc0.1c -C -P -Ba > seno.ps", cpt, xyz)

The point here is that we had to give *cpt, xyz* and not *xyz, cpt* (which would error) because input data associated with an option letter **always comes first** and has to respect the corresponding options order in command string.

To plot text strings we send in the input data wrapped in a cell array. Example:

    lines = Any["5 6 Some label", "6 7 Another label"];
    gmt("pstext -R0/10/0/10 -JM6i -Bafg -F+f18p -P > text.ps", lines)

and we get back text info in cell arrays as well. Using the *G* grid computed above we can run *gmtinfo* on it

    info = gmt("gmtinfo", G)

At the end of an **GMT** session work we call the internal functions that will do the house keeping of freeing no longer needed memory. We do that with this command:

    gmt("destroy")

Sure, we could as well have shut down Julia but calling *gmt("destroy")* serves also another purpose that at times may be quite important. What it does is to clear up the **GMT** session and remove it. We may want to force this when we call *gmt("gmtset")*. The subtle thing about it is that if we have a session running, calling *gmtset* wont do anything because that command is ignored. Only upon starting a session will the contents of *gmt("gmtset ...")* be taken into account.

So that's basically how it works. When numeric data has to be sent *in* to **GMT** we use Julia variables holding the data in matrices or structures or cell arrays depending on the case. On return we get the computed result stored in variables that we gave as output arguments. Things only complicate a little more for the cases where we can have more than one *input* or *output* arguments. The file *gallery.jl*, that reproduces the examples in the Gallery section of the GMT documentation, has many (not so trivial) examples on usage of the **GMT** wrapper.

----------

The  Grid type
-------------

    type GMTJL_GRID 	# The type holding a local header and data of a GMT grid
	   ProjectionRefPROJ4::ASCIIString
	   ProjectionRefWKT::ASCIIString
	   range::Array{Float64,1}
	   inc::Array{Float64,1}
	   n_rows::Int
	   n_columns::Int
	   n_bands::Int
	   registration::Int
	   NoDataValue::Float64
	   title::ASCIIString
	   remark::ASCIIString
	   command::ASCIIString
	   DataType::ASCIIString
	   x::Array{Float64,1}
	   y::Array{Float64,1}
	   z::Array{Float32,2}
	   x_units::ASCIIString
	   y_units::ASCIIString
	   z_units::ASCIIString
    end
