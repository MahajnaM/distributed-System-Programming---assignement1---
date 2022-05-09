Hanna Hayik 207442054 + Mahmoud Mahajne 315256362

Execution Instructions:
	1. Compile the source files through eclipse by specifying Maven goals as: "clean compile package"
		or by running Maven from cmd as: mvn compile package
	2. Go to your Home directory (ex: in Windows "C:\\Users\\Hanna"), Create a directory called ".aws" and
		create a file called "credentials"
	3. Open "credentials" and paste into it the AWS credentials from ur lab session.
		NOTE: everytime you restart the lab, the credentials must be reloaded into this file.
	4. Put the input file in the same directory with the JAR file
	5. Put englishPCFG.ser.gz module in the same directory with the JAR file
	6. From CMD run: "java -jar Local-1.0-SNAPSHOT.jar input-sample.txt output.html 1 terminate"
		NOTE: check Manager/main.java for more execution options

Technical Info:
	- AMI: ami-00e95a9222311e8ed (Amazon Linux 64bit)
	- Instance type: T2.Medium (there is an explanation for this choice)
	- Execution time: 2006 seconds (33.4 minutes) on 9 files (Moodle input, 9 parsing operations) with 1:1 workerToFile ratio
			  3252 seconds (54.2 minuts) on 9 files (Moodle input) with 1:2 worker to file ratio

Workflow (simplified):
	1. Local starts Manager instance if not started yet
	2. Local checks/creates the persistant buckets (JAR bucket, Results bucket) if they exists, has access
	3. Local checks/uploads if the files are there (parser module englishPCFG.ser.gz, assignment JAR...)
	4. Local sends SQS message to the Manager notifying it about a new job
	5. Local now awaits for a "finished" message from Manager
	6. Manager creates proper buckets/queues for communications
	7. Manager initaites a proper number of Workers
	8. Manager creates an SQS message for every line in the input & sends to workers
	9. Manager now awaits for the results from workers
	10. Workers try to read messages from proper queues
	11. Workers finds a job, downloads a file, parses it, uploads results to S3 and notifies Manager about the results
	12. Manager gathers the results for the same Local in one file and uploads it to S3
	13. Manager notifies Local about finishing the job & the location of the results file in S3
	14. Local downloads the results file, extracts the needed info to create the HTML file

Critical Points:
	-Did you think for more than 2 minutes about security? Do not send your credentials in plain text!
	==> Credentials are parsed from the PC and sent to AWS services directly, nothing to encode/encrypt in our app

	-Did you think about scalability? Will your program work properly when 1 million clients connected at the same time?
	==> Yes it will work, (disregard the 19 instances limit for lab users :P) not much threads in our app, synchronized as it should, no waiting times (other than waiting for results)
		minimal use of buckets/queues, so IMO it should work perfectly too

	-What about persistence? What if a node dies? What if a node stalls for a while? Have you taken care of all possible outcomes in the system?
	==> We have taken in regards many things: including errors while: dowloading the files, parsing, sending messages, IO exceptions, and we covered all aspects of this matter.
		Plus our Manager regularly checks on the # of Workers to make sure they match the job, so if any node dies it will be replaced, if it dies mid-job, another Worker
		will take over the job instead (when he's free)

	-Threads in your application, when is it a good idea? When is it bad? Invest time to think about threads in your application!
	==> Manager uses a threadpool for execution, and the Workers has 2 threads: one parses the file and other is created to ChangeVisibility of the recieved message & finally deletes it.
		I think we chose good places to use Threads in the task.

	-Did you run more than one client at the same time? Be sure they work properly, and finish properly, and your results are correct
	==> Yes we did, both finished, checked the output files also, all looked good.

	-Do you understand how the system works? Do a full run using pen and paper, draw the different parts and the communication that happens between them.
	==> Yes of course we do, we even illustrated a sketch of the workflow on Windows Painter :)

	-Did you manage the termination process? Be sure all is closed once requested!
	==> Every Class (Manager/Worker/Local) that creates a resource (Instance/Bucket/Queue) was made sure to close it when done, and we can prove that by using the CLI after the
		application terminates.
	
	-Did you take in mind the system limitations that we are using? Be sure to use it to its fullest!
	==> 19 instances limit was taken into account, all Workers are made sure to work to the full potential also

	-Is your manager doing more work than he's supposed to? Have you made sure each part of your system has properly defined tasks? Did you mix their tasks? Don't!
	==> NO, our manager handles incoming messages from Locals by splitting them into tasks for workers and finally replying to Locals with the results, nothing too much here.

	-Lastly, are you sure you understand what distributed means? Is there anything in your system awaiting another?
	==> Yes we do, other than Locals waiting for results, our system is always busy and we always thrived to use it to the fullest. We also have only 1 "synchronized" word in our code
		meaning we made sure everything is done securely & fast as possible.
	