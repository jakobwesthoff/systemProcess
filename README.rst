=============
SystemProcess
=============

A current project of mine demanded me to invoke a lot of different programms
on the commandline. Some calls were simple executions of a command followed by
one or two arguments. Others were more complex with the need to define special
environment variables or custom file descriptors. Some of them even needed me
to asyncronously run the programs to be able to read their output during
execution or run multiple commands at once. Because nobody, includind me,
really likes the proc_* family of functions like proc_open in php, I decided
to write a simple and yet powerful object oriented wrapper around all this
functionality. The wrapper is fully unit tested and licensed under LGPLv3 for
you to use in your projects.

Features
========

The wrapper currently has the following features:

- Automagically escaping of commands and arguments
- Redirecting stdout/stderr output
- Create multiple commands and pipe their output to each other
- Set custom environment variables
- Set a custom working directory
- Provide custom stdin data 
- Use an arbitrary amount of custom file descriptors
- Execute commands syncronously or asyncronously
- Send signals to running processes

This is are all currently supported features. I think nearly all of the
usecases of invoking an external command are covered by these. If there is any
functionality or feature I have not yet thought of, please drop me a line and
I will be happy to add it. 


Further information and examples
================================

SystemProcess was designed having great flexibility combined with a maximum
amount of comfort in mind.  The fluent interface pattern is used to provide an
easy and readable way of defining complex commandstrings as well as simple
ones. There is no need to handle the escaping of your arguments as this will
be done automatically.

The constructor takes the executable to run as an argument. The following
example will execute the command "echo" with the two arguments "foo" and
"bar"::

    <?php
    $p = new pbsSystemProcess( 'echo' );
    $p->argument( 'foo' )->argument( 'bar' );
    $returnCode = $p->execute();
    ?>

As you can see the fluent interface is used to combine the argument calls in
a readable way.

Quite complex constructs containing redirects, pipes or even custom
file descriptors are possible too. They can be realized with nearly no
effort::

    <?php
    $consumer  = new pbsSystemProcess( 'cat' );
    $consumer->redirect( pbsSystemProcess::STDOUT, pbsSystemProcess::STDERR );
    
    $provider  = new pbsSystemProcess( 'echo' );
    $provider->nonZeroExitCodeException = true;
    $provider->argument( 'foobar' )
             ->pipe( $consumer )
             ->execute();

    var_dump( $provider->stderrOutput );
    ?>

As you can see even complex commands are still quite readable. If the
attribute "nonZeroExitCodeException" is set to true an exception will be
thrown instead of just returning a non zero exit code. This exception will
contain the stdout- and stderrOutput as well as the executed command string.

In case you need asyncronous execution call the execute function with the
first argument set to "true". You will get a set of pipes in return which you
can work with like any other stream in php.

If you just want to use this classes api to generate the shell commands, but do
have no intention to actually execute it you can use the __toString()
functionallity of SystemProcess. An explicit conversion of this object to
string will give you the string context as well, as a use in any string context
like printf. ::

	<?php
	$p = new pbsSystemProcess( 'echo' );
	$p->argument( 'foo' )
	  ->argument( 'bar' )

	// Store command to a variable
	$command = (string)$p;
	// Or print it out
	echo $p, "\n";
	?>

For the more advanced functionallity like sending signals to running processes
take a look at the api documentation, which is well structured und quite
complete.
