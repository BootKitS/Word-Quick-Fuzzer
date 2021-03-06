Word Quick Fuzzer v 1.0
=======================

    $$\      $$\           $$$$$$$$\ 
    $$$\    $$$ |          $$  _____|
    $$$$\  $$$$ | $$$$$$\  $$ |      
    $$\$$\$$ $$ |$$  __$$\ $$$$$\    
    $$ \$$$  $$ |$$ /  $$ |$$  __|   
    $$ |\$  /$$ |$$ |  $$ |$$ |      
    $$ | \_/ $$ |\$$$$$$$ |$$ |      
    \__|     \__| \____$$ |\__|      
                       $$ |          
                       $$ |          
                       \__|          
                   

Word Quick Fuzzer is a [Microsoft Quick Fields list](https://support.office.com/en-us/article/list-of-field-codes-in-word-1ad6d91a-55a7-4a8d-b535-cf7888659a51) fuzzer aiming for Word Quick Fields, written in Python. 


Motivation
=========================================
There has yet to be a fuzzer aiming specifically for the Word Quick Fields. While this is some what of a niche, these fields pose a big risk as we've seen in DDE attacks and alike.

Dependencies
==================
MqF is written and tested on Python v2.7. it uses the following third party libraries

1. [winappdbg](https://github.com/MarioVilas/winappdbg)


2. [pyautoit](https://pypi.python.org/pypi/PyAutoIt/0.3)

3. [Domato](https://github.com/google/domato)

Since we feed random yet valid data into target application during fuzzing, target application reacts in many different ways. During fuzzing the target application may throw different errors through different pop-up windows. To continue the fuzzing process, the fuzzer must handle these pop-up error windows properly. MqF uses PyAutoIT to suppress different application pop-up windows. PyAutoIt is Python binding for AutoItX3.dll

				

Adding More POPUP / Errors Windows Handler
===============================================

The default PopUpKiller.py file provided with Word Quick Fuzzer, is having few most occurred pop up / error windows handler for MS Word, MS Excel & Power Point. Using AutoIT Window Info tool (https://www.autoitscript.com/site/autoit/downloads/) you can add more POPUP / Errors Windows Handlers into 'PopUpKiller.py'.

![Example popup](https://github.com/VotiroLabs/Word-Quick-Fuzzer/blob/master/popuphandler.PNG)
So to be able to Handle the error pop up window shown in screen shot, following lines need to be added in : PopUpKiller.py

```python
if "PowerPoint found a problem with content"  in autoit.win_get_text('Microsoft PowerPoint'):
	autoit.control_click("[Class:#32770]", "Button1")

```

How it works
=============
MqF launches Word with this document and then starts updating its Quick field using COM Update method.
- The reason I chose this method is that it enables to only load Word once, saving a lot of process time. 
- The main thing to realize is that unlinke fuzzers that launch and close the program on each input, in MqF Word remains open and the document remains unchanged - the only thing that changes is the symbolic link, pointing to a new input each time.
- The provided files contains an auto-updating Quick field, pointing to a symbolic link in the same directory. 
- For best performance, make sure to disable "Safe mode" and "Protected View". At the very least, make sure the document is "trusted", so it won't open in "Protected View" - This can be achieved by opening it once and "Enabling Content" or editing it's ZoneIdentifier.

If a crashing input was found, it will be copied for furhter analysis and Word will be relaunched to continue fuzzing.
When all input files have been tested, we move on to the crash analysis phase where Word will be launched per crashing file, testing the crash and recording what happened.
It is at this stage that the "autoupdating" link becomes important - the PopUp module is configured to click "yes" on the autoupdate prompt, effectively updating the link causing Word to load the crashing input file.



Execution
===================
This fuzzer is tested on 32 Bit and 64 Bit Windows Platforms (32 Bit Office Process). All the required libraries are distributed with this fuzzer in 'ExtDepLibs/' folder.

Make sure you execute this in an administrator mode, so python will be able to create symbolic links!

~~~~
python fuzzHTML.py


    $$\      $$\           $$$$$$$$\
    $$$\    $$$ |          $$  _____|
    $$$$\  $$$$ | $$$$$$\  $$ |
    $$\$$\$$ $$ |$$  __$$\ $$$$$\
    $$ \$$$  $$ |$$ /  $$ |$$  __|
    $$ |\$  /$$ |$$ |  $$ |$$ |
    $$ | \_/ $$ |\$$$$$$$ |$$ |
    \__|     \__| \____$$ |\__|
                       $$ |
                       $$ |
                       \__|

    MSWORD Quick Fields Fuzzing Framework.
        Author : Amit Dori (twitter.com/_AmitDori_)

usage: HTMLfuzzer [-h] [-w WORD_FILE] [-r REF_FILE] [-n NUMBER_OF_FILES]
                  [-i INPUTS_DIR] [-o OUTPUT_DIR] [-v] [-d]
                  {analyze,fuzz}

~~~~

Basically, there are 2 modes of operation: analyze OR fuzz. This is the only required argument to the program.

The rest are predifined (but can be user supplied as well):
- -w WORD_FILE: which Word file to use, could be any format that Word can parse.
- -r REF_FILE: what will be the name of the symbolic link file. 
- -n NUMBER_OF_FILES: number of HTML files to be generated by DOMATO.
- -i INPUTS_DIR: directory to save generated HTML files.
- -o OUTPUT_DIR: directory to save crashing HTML files for further analysis.
- -v: verbose mode (logger on DEBUG).
- -d: delete inputs at exit.

___________________________________________________________________________________

~~~~
python fuzzPics.py


    $$\      $$\           $$$$$$$$\
    $$$\    $$$ |          $$  _____|
    $$$$\  $$$$ | $$$$$$\  $$ |
    $$\$$\$$ $$ |$$  __$$\ $$$$$\
    $$ \$$$  $$ |$$ /  $$ |$$  __|
    $$ |\$  /$$ |$$ |  $$ |$$ |
    $$ | \_/ $$ |\$$$$$$$ |$$ |
    \__|     \__| \____$$ |\__|
                       $$ |
                       $$ |
                       \__|

    MSWORD Quick Fields Fuzzing Framework.
        Author : Amit Dori (twitter.com/_AmitDori_)

usage: ImageFuzzer [-h] [-w WORD_FILE] [-r REF_FILE] [-i INPUTS_DIR]
                   [-o OUTPUT_DIR] [-v] [-d]
                   {analyze,fuzz} inputs_dir
~~~~

Basically, there are 2 modes of operation: analyze OR fuzz.
In addition, an inputs folder needs to be defined - Test this with [AFL generated images](http://lcamtuf.coredump.cx/afl/demo/).

The rest are predifined (but can be user supplied as well):
- -w WORD_FILE: which Word file to use, could be any format that Word can parse.
- -r REF_FILE: what will be the name of the symbolic link file. 
- -o OUTPUT_DIR: directory to save crashing image files for further analysis.
- -v: verbose mode (logger on DEBUG).
- -d: delete inputs at exit.


Since MqF supports custom user files, you will be needing to take care to these features:
1. create a document in Word, insert a Quick field of type INCLUDETEXT/INCLUDEPICTURE and point it to a non-existant html/image file path.
- This non-existant file path should be provided to MqF in the "-r" argument, as it will be the generated symbolic link.
2. Open the document with an archive unpacker and edit "word\document.xml": look for a tag containing "begin" and add w:dirty="true" -> it should look like this: `<w:fldChar w:fldCharType="begin" w:dirty="true"/>`
3. Exit the text editor and make sure the edits are saved to the document. now you can supply its path to MqF in the "-w" argument.


Few More Points about Word Quick Fuzzer:
======================================
1. Fuzzing Efficiency:
To maximize fuzzing efficiency, Word is loaded once and the refresh is made using COM and symbolic links. Once crashed, Word will be relaunched and the process continues.

2.  Hybrid approach:
At the moment, the fuzzer is hybrid in the sense that it first generates all inputs and then feeds them into Word, keeping track of crashing inputs. Only when all inputs have finished, it moves forward to analyze each crash.
It operates in this way as I was having some issues utilizing a debugger alongside COM control, so we got COM control (at the input testing stage)  -> debugger control (at the crash analysis stage).


TODO
=======
1. TODO: timeout mechanizm for the COM Update?
2. ISSUE: A race condition occurs sometimes between COM and the debugger, causing the "Update" to do nothing. It's somehow related to Word loosing focus.
3. TODO: create a "not-responding" guarding thread.
4. TODO: Add workers for HTML/Image generation and feeding the queue. continous fuzzing.
5. TODO: Add injection of refFile to supplied wordFile.
6. TODO: Improve HTML generation [incorporate in fuzzer process or continue in seperate process with different configuration].



Author
=============
[Amit Dori](https://twitter.com/_AmitDori_), Security Researcher at Votiro.


Inspiration for this tool
=========================
- Based on [OpenXMolar]( https://github.com/debasishm89/OpenXMolar) by [Debasish Mandal](https://twitter.com/debasishm89)
- [Domato](https://github.com/google/domato) by [Ivan Fratric](ifratric@google.com)