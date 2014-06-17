How is beam file loaded into Erlang VM
=======
I have been using erlang for quite long time but i have been always curious about how the Erlang binary is loaded in to Erlang VM. Regardless the standard Erlang beam files, i know it's a symbolic assemble, what about native codes?
Anyway i will take some time and look into OTP and post my findings here. 
This file will be updated accordingly. I am quite busy on works -- actualy i am so busy and i have so many things want to do, so i don't know when will this be done. Personally i hope I can find the answer within 1 week.  

I cannot find much from the erlang documentation but this:
>The object code must be loaded into the Erlang runtime system. This is handled by the code server, see code(3).
>
>The code server loads code according to a code loading strategy which is either interactive (default) or embedded. In interactive mode, code are searched for in a code path and loaded when first referenced. In embedded mode, code is loaded at start-up according to a boot script. This is described in System Principles.

Well, this explains something but basically useless. 
Let's dig into the code. 

c.erl
----
c.erl is the cover used for shell for some useful commands. 
During my work, i use nl(Module) a lot. I know the shell is actually calling the c:nl(Module).
Look into the cover module, the code is looking like below:
>nl(Mod) ->
>    case code:get_object_code(Mod) of
>	{_Module, Bin, Fname} ->
>            rpc:eval_everywhere(code, load_binary, [Mod, Fname, Bin]);
>	Other ->
>	    Other
>    end.

so code:load_binary is actually called. 

code.erl
-----
the code module simply translate the reqest into code_server:call(Request).

code_server.erl
-------
It's becoming interesting. the code_server seems to be a server which handling all kinds of code requests. 
The code_server is started from the kernel application. During the init/0 of kernel.erl, the code_server is started and supervised by the kernel_sup.
use appmon:start() and we can see the kernel_sup supervision tree. 

THe following is a snampshot:
![Kernel Supervision Tree] (./imgs/kernel_app_tree.png)

code_server initialization during boot
--------
first, the code_server will register itself as code_server. 
Then, the code_server created a ets table and fills it with preloaded modules and init:fetch_loaded()

init:fetch_loaded()
------
This function is just returning the loaded attribute of the internal #state. 
The interesting question is: how are those modules are fetched into this #state. 




