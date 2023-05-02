Download Link: https://assignmentchef.com/product/solved-cmpt433-assignment-4-linux-drivers
<br>



Submit deliverables to CourSys: <a href="https://courses.cs.sfu.ca/">https://courses.cs.sfu.ca/ </a>This assignment is to be done <strong>individually or in pairs</strong>.

Post questions to Piazza discussion forum.

Do not give your work to another student/group, do not copy code found online, and do not post questions about the assignment to other online forums.  See the marking guide for details on how each part will be marked.

<h1>1.   Morse Code Driver</h1>

In this assignment you will create a Linux kernel driver which will flash Morse code on the Zen cape’s LEDs.

<strong>Morse Code (see </strong><a href="https://en.wikipedia.org/wiki/Morse_code"><strong>Wikipedia</strong></a><a href="https://en.wikipedia.org/wiki/Morse_code"><strong>)</strong></a><strong>:</strong>

<ul>

 <li>Morse code is a way of transmitting text (such as a message “Hello World”) as short and long beeps, or in our case, short and long flashes on an LED. The short beeps/flashes are called “dots”, and the long ones called “dashes”.</li>

 <li>A “dot” is the basic unit of time. For now, assume the “dot” time is 200 ms.</li>

 <li>A “dash” is three dot-times long.</li>

 <li>Each dot/dash is separated by one dot-time. During this time the LED is off.</li>

 <li>Two letters in a message are separated by three dot-times. During this time the LED is off.</li>

 <li>Spaces between words are equal to seven dot-times total (no additional 3 dot-time intercharacter delay).</li>

 <li>See the Wikipedia link for more explanation of timing.</li>

 <li>Morse code does not differentiate between upper and lower case letters.</li>

</ul>

<h2>1.1    General Requirements</h2>

<ol>

 <li>All printk()’s must use reasonable log-levels (KERN_INFO, KERN_ERR,…)</li>

 <li>Your driver must correctly load and unload when using insmod and rmmod.</li>

</ol>

<ul>

 <li>It should clean-up after itself so you can re-load it often.</li>

</ul>

<h2>1.2    Create a ‘Misc’ Driver to Flash Morse Code</h2>

<ol>

 <li>Use the driver-creation guide to create a skeleton driver named morsecode.

  <ul>

   <li>Setup the driver to build into a file named ko</li>

   <li>In your makefile, when building the driver, automatically copy or build the driver to</li>

  </ul></li>

</ol>

~/cmpt433/public/drivers/

<ul>

 <li>Your makefile must work with any user ID (don’t hard code your ID!). You can use ~ to refer to the home directory. It must build on a Linux PC that has been setup in accordance with the driver creation guide.</li>

</ul>

<ol start="2">

 <li>Make your driver’s init and exit methods display a message using printk().</li>

 <li>Make your driver a misc-character driver which supports a read and write method.

  <ul>

   <li>Have it setup its device virtual file (node) as /dev/morse-code</li>

  </ul></li>

 <li>Add a new LED trigger named “morse-code”.

  <ul>

   <li>Register the trigger at the driver’s initialization, and unregister at exit.</li>

  </ul></li>

 <li>Implement the write() method as follows:

  <ul>

   <li>Make it a blocking call (do not return until all flashing has completed).</li>

   <li>Flash the message to the LEDs using the kernel’s LED library and the morse-code trigger you created.</li>

   <li>You’ll need to loop through all characters in the input buffer, determining the flash code for each letter, and then flashing it out to the LED(s).</li>

   <li>The course website provides an encoding for each letter. <strong>You must use these.</strong></li>

   <li>Any character not a space or a letter (a-z, or A-Z) should be skipped with no delay.</li>

  </ul></li>

</ol>

(i.e., ‘Hi There’ should flash the same as ‘!H_I th_ER&amp;<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="92f7d2">[email protected]</a>#09.,-5%!’

<ul>

 <li>You may assume that only one call to your write method will be happening at once so you don’t need to synchronize multiple calls. (If multiple writes do happen concurrently, it will likely inter-mix the flashes from both messages, but that’s OK.)</li>

 <li>You must use copy_from_user() (or similar) to access buffers from user-space.</li>

</ul>

<ol start="6">

 <li>Tips:

  <ul>

   <li>Test using either of the following methods:</li>

  </ul></li>

</ol>

# echo sos &gt; /dev/morse-code or

# cat &gt; /dev/morse-code

<ul>

 <li>In the second one, you can type and when you hit enter, it will send the data. Cancel with Ctrl+C.</li>

 <li>Before any LEDs will flash, you need to set them to the morse-code trigger, much like was done in assignment 1.</li>

 <li>If you rmmod your driver, the morse-code trigger is removed. When you insmod your driver again, you’ll need to reset an LED to be morse-code.</li>

 <li>Consider writing a trivial Linux script to unload the driver, reload the driver, and set the LED trigger to save time when testing new changes to your driver.</li>

 <li>Use the msleep() kernel method (in header linux/delay.h) to suspend for a set number of milliseconds. It’s a blocking write-call so this holds up the calling thread until the write is complete.</li>

 <li>In order for the LEDs to function under your custom kernel, you will likely need to install all the runtime loadable kernel modules you compiled with your kernel. See the Driver Creation Guide, section “Coping Modules to RFS”.</li>

</ul>

<h2>1.3    Driver Parameters</h2>

<ol>

 <li>Add the following driver parameter:

  <ul>

   <li>dottime: Sets the timing of the “dot”, in ms.</li>

   <li>When you change the time for the “dot”, all other flash-timings should change relative to this (as described in the Morse Code section above).</li>

   <li>Default value should be 200ms.</li>

   <li>Value must be limited to [1 – 2000] (inclusive). If out of range, display an error message and set the value to the default.</li>

  </ul></li>

 <li>To configure the parameters at driver load time (from the console) use:</li>

</ol>

# insmod morsecode.ko dottime=150

<ol start="3">

 <li>Tips:

  <ul>

   <li>Make sure all your timings are relative to the “dot” time, which is configurable. This includes the time for “dot”, “dash”, between dots and dashes, and spaces.</li>

   <li>When using the Morse code letter-encoding that is provided, each bit represents one dot-time. Therefore, by setting this time correctly the “dot”, the “dash” and betweendot/dash times are likely to be just one setting (i.e., the dot time).</li>

  </ul></li>

</ol>

<h2>1.4    Read Dashes and Dots via FIFO</h2>

Add a read method to your driver so that it returns the dots and dashes it has transmitted since the last read call.

<ol>

 <li>This method is executed when a program reads from /dev/morse-code, such as:</li>

</ol>

# cat /dev/morse-code

<ol start="2">

 <li>Create a FIFO character queue for the dots, dashes, and spaces being transmitted. This will be populated by your driver’s write function and read in your driver’s read function.

  <ul>

   <li>Each time a dot is flashed out, add a ‘.’ (period) to the queue.</li>

   <li>Each time a dash is flashed out, add a ‘-‘ (minus) to the queue.</li>

   <li>Each break between letters adds one space to the queue.</li>

   <li>Each break between words adds three spaces total to the queue.</li>

   <li>At the end of transmitting a message, add a line-feed (‘
’) to the queue.</li>

   <li>For example, “Hi me” generates the following</li>

  </ul></li>

</ol>

…. ..   — .


<ul>

 <li>Below is the same output, but with additional annotations for additional understanding (your program does not produce this annotated output):</li>

</ul>

<table width="538">

 <tbody>

  <tr>

   <td width="538"><strong>…. ..   — .
</strong>     <em>(dot’s and dashes, and line-feed) </em>H    i    m  e       <em>(letters in the message)</em>s  sss  s        <em>(s = space in dashes and dots message)</em></td>

  </tr>

 </tbody>

</table>

<ol start="3">

 <li>Make the read function non-blocking: When called, any data waiting in the queue is returned to the caller but if there is no data available then it immediately returns 0 bytes read.

  <ul>

   <li>Each character read from the queue by the read function removes that data from the queue. You must safely access the buffer from user space using something like copy_to_user().</li>

   <li>You must correctly implement the case where the read buffer is smaller than the amount of data waiting to be read. Hint: Use the readfile program on the course website to test. If using the kernel queue data structure, it may be trivial to correctly implement this.</li>

  </ul></li>

 <li>Examples:

  <ul>

   <li>Simple SOS pattern (easy to recognize). Note that calling cat twice generates no output the second time because the queue is empty.</li>

  </ul></li>

</ol>

<table width="538">

 <tbody>

  <tr>

   <td width="538"># <strong>echo ‘SOS’ &gt; /dev/morse-code</strong># <strong>cat /dev/morse-code </strong>… — …# <strong>cat /dev/morse-code</strong>#</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>Adding spaces between the letters means the output has three spaces between the words instead of one space (as above) between the letters.</li>

</ul>

<table width="538">

 <tbody>

  <tr>

   <td width="538"># <strong>echo ‘S O S’ &gt; /dev/morse-code</strong># <strong>cat /dev/morse-code </strong>…   —   …</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>The letters E I S H are just dots (1 through 4); and T M O are dashes (1 through 3), which makes the following easy to debug:</li>

</ul>

# <strong>echo ‘e i s h t m o  eishtmo’ &gt; /dev/morse-code</strong>

# <strong>cat /dev/morse-code </strong>

.   ..   …   ….   –   —   —   . .. … …. – — —

<ul>

 <li>And, of course:</li>

</ul>

# <strong>echo ‘Hello world!’ &gt; /dev/morse-code</strong>

<strong># cat /dev/morse-code </strong>…. . .-.. .-.. —   .– — .-. .-.. -..

<ul>

 <li>A longer output:</li>

</ul>

<table width="538">

 <tbody>

  <tr>

   <td width="538"># <strong>cat &gt; /dev/morse-code </strong>But soft! What light through yonder window breaks?# <strong>cat /dev/morse-code </strong>-… ..- –   … — ..-. –     .– …. .- –   .-.. .. –.…. –   – …. .-. — ..- –. ….   -.– — -.-.. . .-.   .– .. -. -.. — .–   -… .-. . .- -.- …</td>

  </tr>

 </tbody>

</table>

<ol start="5">

 <li><strong>Test real-time output with two terminals open:</strong></li>

</ol>

Terminal 1: Display, in real time, what is being output from /dev/morse-code:

# for((;;)) do cat /dev/morse-code; sleep 0.1; done;

This will show you the ‘.’ and ‘-‘ characters as they are generated by your driver!

Terminal 2: When you want to test, echo new text into /dev/morse-code:

# echo ‘hello world!’ &gt; /dev/morse-code

<ol start="6">

 <li>Tips:</li>

</ol>

<ul>

 <li>Assume there is at most one write and one read happening at the same time.</li>

 <li>If a read happens part-way through flashing out a transmission, it will return the dashes and dots that have been flashed so far; further reads will return the later dashes and dots. This is by design.</li>

 <li>While your write routine is flashing out dots and dashes, you’ll want to interpret the bits to detect dots and dashes to put ‘.’, ‘-‘, and ‘ ‘ characters into the queue.</li>

 <li>Count the number of dashes and dots by incrementing a counter whenever one is detected.</li>

 <li>When a dash, a dot, an inter-character break, an inter-word break, or end of transmission is detected, push the correct character into the queue for the read function to retrieve.</li>

 <li>You can detect a dash with the bit pattern: 1110 (four msb’s).</li>

 <li>You can detect a dot with the bit pattern: 10 (two msb’s). However, if you are shifting the Morse-code bit pattern to the left each time step, a dash when shifted left two will look like a dot. Therefore you may need to add a cool-down feature to prevent detecting spurious dot’s. (i.e., once you detect a dash, don’t detect a dot for a certain number of bit-times).</li>

 <li>There is a function for kfifo which works with user-space buffers.</li>

</ul>

<h2>1.5    Flash on Zen LED &amp; Capture Output</h2>

On the <strong>target</strong>, boot into your custom compiled Linux kernel (downloaded using UBoot) and capture the console interaction of executing the following commands. Save the capture into a file named capture.txt:

<ol>

 <li><em>First ensure you can load the Zen cape’s LED support, as shown in the ZenLEDGuide. </em></li>

 <li>Display the kernel version:</li>

</ol>

# uname -r

<ol start="3">

 <li>Display the booted kernel’s command line:</li>

</ol>

# cat /proc/cmdline

<ol start="4">

 <li>Display target’s networking configuration (must show Ethernet because it’s built into the kernel):</li>

</ol>

# ifconfig

<ol start="5">

 <li>Load the Zen cape’s LED virtual cape (see guide).</li>

 <li>Run modinfo on ko.</li>

 <li>Load the Morse code driver (insmod), <strong>selecting a dot time of 40 ms.</strong></li>

 <li>List loaded modules (lsmod).</li>

 <li>Set the Zen cape’s red LED to be driven by the morse-code</li>

 <li>Display the list of triggers for the Zen cape’s red:</li>

</ol>

# cat /sys/class/leds/zencape:red/trigger

<ol start="11">

 <li>Flash out the message Hello world!:</li>

</ol>

# echo ‘Hello world.’ &gt; /dev/morse-code

<ol start="12">

 <li>Display the dash-dot flash codes:</li>

</ol>

# cat /dev/morse-code

<ol start="13">

 <li>Remove the module (rmmod).</li>

 <li>List loaded modules (lsmod).</li>

 <li>Display all the kernel messages your driver output:</li>

</ol>

# dmesg | tail -100

<ul>

 <li>Delete all dmesg lines not from your driver just to keep the output smaller.</li>

</ul>

Include in capture.txt any notes to the marker about how to get your code to work if there are any unusual steps (you may assume the marker has completed the Zen Cape LEDs guide and the Driver Creation Guide. You need not include the output from UBoot.

Make sure your capture file is human readable! If there are any funny characters (like ^D or [[^C] please edit the file to make its contents clearer. (These can arise due to colours in the console). An easy way to capture this text is execute the commands via SSH, and then copy-and-paste the contents of the SSH session into a text file.

<h1>2.   LED Wiring</h1>

Wire up one of the large discrete (loose component) LEDs included with your BeagleBone Green kit into the breadboard for being safely controlled by software. You may choose any colour LED. Then registered it as an LED with the kernel, which makes it accessible inside the /sys/class/ leds/ folder.

<strong>Hint:</strong> Carefully follow the Wiring An LED guide.

<strong>Warning:</strong> Incorrectly wiring the circuit could damage your BeagleBone Green. I strongly recommend you have another person (such as a partner on the assignment, or another student in the class) double check your circuit before applying power.

Record a video of you controlling the LED via the Linux command line:

<ol>

 <li>Must register it as an LED with the kernel (you need not show this in the video).</li>

 <li>Show the breadboard with the wired LED circuit.</li>

 <li>Show the Linux terminal where you enter the command to turn on the LED.</li>

 <li>Show the LED that has turned on.</li>

 <li>Show the Linux terminal where you enter the command to turn off the LED.</li>

 <li>Show the LED that has turned off.</li>

 <li>Show the Linux terminal where you enter the command to set the LED to the heartbeat trigger.</li>

 <li>Show the LED turning on and off for ~10 seconds.</li>

</ol>

Upload your video to some place it will be accessible to the TA, such as YouTube, your SFU Vault drive (share file with TA), Vimeo, …. Video must be less than 5 minutes long! Shorter is better!


