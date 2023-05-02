Download Link: https://assignmentchef.com/product/solved-cop4600-project-1-system-calls
<br>
<h1>[pdf-embedder url="https://assignmentchef.com/wp-content/uploads/2022/12/Project-1-System-Calls.pdf" title="Project 1 - System Calls"]</h1>
<h1>Prologue</h1>
You work for a top-secret shadow government organization dedicated to the rise of the Silurian overlords. You, as a faithful member of the Lizard Legion, are part of the team charged with improving data storage and handling, particularly tracking <strong><em>metadata</em></strong> – that is, data about data – within the organization’s computer systems. You have been tasked to build a coded message subsystem under the guise of process logging for kernels running in “Sky Skink”, the cloud computing system. Naturally, the Legion uses the superior Reptilian operating system distribution.
<h1>Overview</h1>
In this project, you will implement a system call in Reptilian along with three static library functions that allow the system call to be invoked from a C API. These custom system calls will <em>get</em> and <em>set</em> a custom process log level that will sit atop the standard Linux kernel’s diagnostic message logging system (<strong>dmesg</strong>) and allow processes to submit log entries along with a log level. If the log level for the message is more severe (lower than) the current log level, the message will be added to the kernel log. Log levels and names will correspond to those in the Linux kernel. We, as your benevolent lizard overlords, will provide a program that exercises and demonstrates the new calls. You create a short video to demonstrate your code. (Our masters will be most pleased.) You will submit the project via Canvas so as not to invite suspicion.
<table width="671">
<tbody>
<tr>
<td colspan="4" width="671"><strong>Table 1. Kernel Log Levels and Corresponding Process Log Levels </strong></td>
</tr>
<tr>
<td width="138"><strong>Kernel Level Name </strong></td>
<td width="354"><strong>Description </strong></td>
<td width="36"><strong># </strong></td>
<td width="144"><strong>Process Level Name </strong></td>
</tr>
<tr>
<td width="138">KERN_EMERG</td>
<td width="354">Emergency / Crash Imminent (no process logging)</td>
<td width="36">0</td>
<td width="144">PROC_OVERRIDE</td>
</tr>
<tr>
<td width="138">KERN_ALERT</td>
<td width="354">Immediate Action Required</td>
<td width="36">1</td>
<td width="144">PROC_ALERT</td>
</tr>
<tr>
<td width="138">KERN_CRIT</td>
<td width="354">Critical/Serious Failure Occurred</td>
<td width="36">2</td>
<td width="144">PROC_CRITICAL</td>
</tr>
<tr>
<td width="138">KERN_ERR</td>
<td width="354">Error Condition Occurred</td>
<td width="36">3</td>
<td width="144">PROC_ERROR</td>
</tr>
<tr>
<td width="138">KERN_WARNING</td>
<td width="354">Warning; recoverable, but may indicate problems</td>
<td width="36">4</td>
<td width="144">PROC_WARNING</td>
</tr>
<tr>
<td width="138">KERN_NOTICE</td>
<td width="354">Notable, but not serious (e.g., security events)</td>
<td width="36">5</td>
<td width="144">PROC_NOTICE</td>
</tr>
<tr>
<td width="138">KERN_INFO</td>
<td width="354">Informational (e.g. initialization / shutdown)</td>
<td width="36">6</td>
<td width="144">PROC_INFO</td>
</tr>
<tr>
<td width="138">KERN_DEBUG</td>
<td width="354">Debug messages</td>
<td width="36">7</td>
<td width="144">PROC_DEBUG</td>
</tr>
</tbody>
</table>
<strong>NOTE: Take snapshots of your VM! You will probably brick your machine at some point during this or other projects, and you will not want to start from scratch. <u>No, seriously – take snapshots!</u> </strong>
<h1>Structure</h1>
The project is broken into four main parts:
<ul>
 	<li>Create a kernel-wide <em>process log level </em></li>
 	<li>Create system calls that allow a process to <em>get</em> or <em>set</em> the <em>process log level </em>of the system.</li>
 	<li>Create system call that allows a process to add a process log message at a defined log level.</li>
 	<li>Create static library functions that allow the system calls to be invoked via a C API.</li>
</ul>
<table width="566">
<tbody>
<tr>
<td width="168">User Program</td>
<td width="29">→</td>
<td width="168">Library</td>
<td width="30">→</td>
<td width="172">System Call</td>
</tr>
</tbody>
</table>
<strong>Figure 1: A system call invoked from a user program </strong>

While exact implementation may vary, the library functions must match the signatures laid out in this document, and the system calls must apply the security model properly. Logged messages have format <strong>“</strong><strong>$log_level_name</strong> <strong>[</strong><strong>$executable</strong><strong>, </strong><strong>$pid</strong><strong>]:</strong> <strong>$message”</strong>, e.g.:

<strong>PROC_ERR [bacon_pancakes, 21]: Life is scary &amp; dark. That is why we must find the light. </strong>
<h2>System Call</h2>
The system will have a single, kernel-wide <em>process log level</em> which should initialize on boot in the kernel and must be stored persistently (until shutdown / reboot). The rules for logging are as follows:
<ul>
 	<li>Any process can <u>read</u> (get) the <em>process log level</em>.</li>
 	<li>Any process may send a process log to the kernel.</li>
 	<li>Only a process running as the <em>superuser</em> may <u>write</u> (set) the <em>process log level</em>.</li>
 	<li>If a message’s log level is higher than the <em>process log level</em>, the message is ignored.</li>
 	<li>If a message’s log level is lower than or equal to the <em>process log level</em>, the message will be logged. 6) The system-wide a <em>process log level</em> should be <u>initialized</u> to zero (0) – i.e., override logging only.</li>
 	<li>Log levels can have values between 0-7 (3-bit unsigned integer). Invalid level results in call failure.</li>
 	<li>Any successfully logged message should be logged with the corresponding kernel log level.</li>
</ul>
System calls are called via syscall(call_num, param1, param2). To log a message, the call should be syscall(PROC_LOG_CALL, msg, level).<em> <strong>Call parameters are limited to no more than <u>two</u>!</strong></em>
<h2>Static Library</h2>
You will create a static library to invoke the system calls in a directory named <strong>process_log</strong>. This include a header, <strong>process_log.h</strong> (prototypes and level symbols), and static library file named <strong>libprocess_log.a</strong><em>.</em> You will also need to provide a Makefile for the library. All sources must be contained within the <strong>process_log</strong> directory. Please note, <em><u>these filenames must match exactly!</u></em>

You will create a tarred gzip file of the <strong>process_log</strong> directory with name <strong>process_log.tar.gz</strong>. When testing, we will decompress the archive, enter the <strong>process_log</strong> directory, and build. All functions enumerated below must be made available by including <strong>” process_log.h”</strong>. See <em>Submission</em> for details.

<u>Library Functions</u>

&nbsp;

<em>int</em> <strong>get_proc_log_level</strong>()

Invokes system call which reads system-wide process log level. Returns the process log level on success, and <strong>-1</strong> otherwise.

<em>int</em> <strong>set_proc_log_level</strong>(int new_level)

Invokes system call which attempts to change the system-wide process log level to <strong>new_level</strong>. Returns <strong>new_level</strong> on success, and <strong>-1</strong> otherwise. On failure, log level should be unchanged.

<em>int</em> <strong>proc_log_message</strong>(int level, char *message)

Invokes system call to log a message for this process. If logged, the message should appear in <strong>dmesg</strong> logs at the corresponding kernel level. Returns <strong>-1</strong> for invalid log level, and <strong>level</strong> otherwise.

<u>Harness Functions</u>

In addition to the standard library functions, you will implement testing harness functions. The testing harness functions are used to verify security of the system calls from the system library (and are required for full credit on this assignment). We will call these functions to retrieve the information needed to make a system call. We will then call the system call within our own program. This ensures that no security checks are being done in the user-level library.

System call parameter retrieval data should be returned as a <u>pointer</u> to an <strong>int</strong> <u>array</u> of 2-4 values that can be used to make the system call (which can be cast from other types). It has this format:

{ call_number, num_parameters [, parameter1] [, parameter2] }

e.g.: { 42, 2, 867, 5309 } à syscall(42, 867, 5309)

<strong>Figure 2: Harness functions can directly invoke system calls without the library functions. </strong>

These test harness elements must be implemented to test your security model:

&nbsp;

<em>#define</em> <strong>PROC_LOG_CALL </strong>&lt;number&gt;

Definition for the system call number for <strong>proc_log_message</strong> (see <strong><em>System Call</em></strong>); should be in header.

<em>int*</em> <strong>retrieve_set_level_params</strong>(int new_level)

Returns an <strong>int</strong> array of 2-4 values that can be used to make the set-process-log-level system call.

<em>int*</em> <strong>retrieve_get_level_params</strong>()

Returns an <strong>int</strong> array of 2-4 values that can be used to make the get-process-log-level system call.

<em>int</em> <strong>interpret_set_level_result</strong>(int ret_value)

After making the system call, we will pass the syscall return value to this function call. It should return <strong>set_proc_log_level</strong>’s interpretation of the system call completing with return value <strong>ret_value</strong>.

<em>int</em> <strong>interpret_get_level_result</strong>(int ret_value)

After making the system call, we will pass the syscall return value to this function call. It should return <strong>get_proc_log_level</strong>’s interpretation of the system call completing with return value <strong>ret_value</strong>.

<em>int</em> <strong>interpret_log_message_result</strong>(int ret_value)

After making the system call, we will pass the syscall return value to this function call. It should return <strong>proc_log_message</strong>’s interpretation of the system call completing with return value <strong>ret_value</strong>.

<u>Note that there is no </u><strong><u>retrieve</u></strong><u> function for </u><strong><u>log_message</u></strong><u> as its system call format is defined above.</u>