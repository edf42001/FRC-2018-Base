This robot project is meant to be the culmination of the best code we have created over the past few years, plus the new code we worked on this offseason. 

Sorry the readme is a little unorganized, as it is simply a collection of the documentation that went into this project. In order:

* Desription of Flexible Autonomous System
* Description of new logging system
* Feature list



About Flexible Autonomous:

Note: The VIs for the flexible autonomous system are stored in Robot-Project/CreateCommands, Robot-Project/Auto, and Support/AutoFileEditor.
 
The flexible autonomous scripting system consists of two main parts: Creation of the XML autonomous file, and execution of the file. During creation, the user uses the mouse and keyboard to script the routine they want. During execution, the robot parses the file to read and execute the autonomous commands. 

Creation:
Open the Support/AutoEditor.lvproj to find the AutoFileCreator VI (only open this VI through the project). The user is greeted with a blank text box. A list of commands is to the left. (For information on creating commands, see section Creating Commands). Two of the options in the box, sequential and parallel, are not commands, but are instead structural elements. Any commands inside a sequential block will be executed one after another, and any in a parallel block will be executed at the same time. Structural elements can be nested for complex behavior. 

To add a command or a structural element to the routine, select it in the command box, then left click the text box where you want to put it. You may only click at the end of a line, right after the closing ">" of a tag to validly add a new command/structure (collectively called tags from now on).

If placing a command, a window will open for you to enter the parameters. (See Creating Commands for how to make these windows). The OK button will confirm what you have entered, while clicking Cancel or closing the window will cancel the placing of the command. 

The routine can be modified by double clicking. A double click will bring up a window with options. You do not have to double click at the end of the line. Current supported operations are Edit, Delete, Copy, Paste and Clear, with thought to add a Move option. Edit allows the user to edit the parameters of the tag. Delete deletes the tag. Clear clears the entire routine. Copy and Paste should be self explanatory. 

There are, of course, the well known Load From Local, Save to Local, and Export to Robot buttons and a selection box with auto files. These do what you would expect.

Clicking ESC unselects the current selected command.

Execution:
The flexible autonomous system is flexible because the user does not need to modify anything for execution to work. It is all based on the available commands given to the system, see Creating Commands. All you have to do is pass it the path on the roboRIO to the file you want to run. Look at ExecuteXML.vi, which is inside of Autonomous.vi, for this code.

Creating Commands:
A "command" is a VI that will execute on your robot. To create a command, add a VI in the special CommandTemplates folder, found under the My Computer section of the robot project. Add controls to the VI for each parameter you want passed to your command. On our robot, these VIs will be wrappers around our Command and Control VIs. When you are done creating or modifying a command template open and run GenerateCode.vi, which will take your templates and convert them into a form that can be run in autonomous. 

Note: Please don't use spaces in your control's names. It won't work. Code may be added later to fix this. 

Currently, only boolean, number, and string parameters are supported. 

Extra Note: Many of the VIs that make up the Flexible Autonomous system have the paths to other folders and VIs as part of their code. If you take the code and intergate it into your own robot project and put the VIs and folders in different locations, you may get File/Folder not found errors. You can go into the VIs with the errors and change the paths. Sorry there is currently no central location for changing these paths.




About new logging:

So basically there's a VI called LoggingOperation that acts as a central server for to-be-logged data. It has a feedback node it uses to store the data. It uses LabVIEW's "variant" (any type) to log any data type you want as log as you manually write the code to convert that data type to a string, which I have done with String, Number, Boolean, and Enum.
It has three operations, Set, GetAll, and GetHeaders
Set takes in the data name and the data, and stores it in the VI.
Get All gets all the stored data and converts it to a csv string.
GetHedears reads the names from the DataNames enum .ctl and outputs the csv string for the headers of the file
All the VIs are in saved in the Robot-Project/Logging folder, and are mostly used in periodic tasks (file writing) and the drive controller (some data writing) for now. 





Feature list:

	Robot-Project/
		Framework/Begin.vi		
			Create MyLogs, Auto, Paths, and AutoFiles folders if they do not already exist on the roboRIO
			Uses Logging/LogOperation.vi to initialize logging “server”
		Framework/Periodic Tasks.vi
			When robot is enabled creates a log file named <date>-<match#> with Logging/CreateFile.vi, then writes the column headers as defined in Logging/DataNames.ctl with Logging/LogHeaders.vi, then logs data in a loop with Logging/LogData.vi, which uses LoggingOperation.vi to write all the data to the file. It automatically adds the time to the first column. The first frame of the flat sequence structure in LogData.vi is where data that doesn’t fit anywhere else should be logged, such as CPU Usage.
		Teleop.vi
			Will need to be modified. Reads joystick data and bundles it into a cluster defined by Configuration/JoystickData.ctl. Modify JoystickData.ctl to add more joystick functionality. This cluster is passed to various <subsystem>Teleop.vi s. Currently there is only
				Drive/DriveTeleop.vi, which drives the robot using CheesyDrive, which is found in ../Dawgma Programing Library/WPI Helper Stuff/CheesyDrive.vi
		Autonomous.vi
			Will need to be modified. Consists of a control that says “Replace me with how you want to select your auto file name”, which is converted to a path and passed to Auto/ExecuteAutoFile.vi, which runs the file.  
		Framework/Disabled.vi
	 	Reset navX gyro on dashboard button press
			Set all subsystems outputs to 0. May want to be modified.  
		Drive/Implementation/Drive Controller.vi
		Initializes a 6 motor drive train, two encoders, and a navX gyro. Will need to be modified
			Sets default values for drive related logging data
			Top loop:
				Reads starting location from dashboard for pure pursuit. Will need to be modified
				Sends gyro and encoder data to dashboard
				Uses ../Dawgma Programming Library/Controls/Pure Pursuit/EncoderGyroToXY.vi  to calculate XY position from encoder and gyro data
				Subtracts off initial value of gyro when auto starts to re-zero. 
				Sends angle, XY position, and encoder velocities to bottom loop with a lossy stream channel for various controllers to use.
				Logs encoder and navX data.
			Bottom loop:
				Implements DriveForDistance, DriveForTime, TurnToAngle, DrivePath, Immediate, and Reserve commands.  These are hopefully going to be encapsulated with a single VI each.
				DrivePath already is, with ../Dawgma Programming Library/Control/Pure Pursuit/PurePursuit.vi
				Hope to replace current implementation of TurnToAngle with motion profiled version
				Logs drive motor values and current command
		CreateCommands/
			Contains code for creating auto commands. The folder /CommandTemplates is underneath the My Computer section of the robot project, and is where you put all command templates. After creating templates run /GenerateCode.vi, also under My Computer, to convert the templates in executable autonomous VIs, which are put into Auto/Commands/. It also generates Auto/ExecuteCommand.vi which calls the different commands in auto. 
			Four auto commands are currently implemented:
				DriveForTime, which makes the robot drive at a speed for a time
				DriveForDistance, which makes the robot drive for a distance at a speed, at a certain angle which is currently the last angle it turned to. This needs to be modified so the angle can also be given in the auto file editor.
				TurnToAngle, makes the robot turn to an angle and stores the angle in a global variable so DriveForDistance knows which angle to drive at. 
				Wait, a pause
		../Dawgma Programming Library/Controls/Pure Pursuit/TestPurePursuit.vi and Logging/LoggingTesting.vi are also under My Computer for testing. 
	Dashboard-Project/Dashboard Main.vi
		Sends date string to robot for log file name
		Displays gyro and encoder values in Drive tab, with reset button for gyro
		No auto selectors are set up, will need to be modified. 
		To use enums on dashboard modify the VI “auto bind dashboard controls” or something like that. This bridge will be crossed later if we want.
		Sends battery number for log file. 
	Support/AutoEditor.lvproj
		Contains two VIs:
			 GenerateXML.vi, the interface for creating auto files. Uses the VIs in ../Robot-Project/CreateCommands/CommandTemplates/ for the list of commands you can use in you file and for entering parameters to your file. Has save, load, and export capabilities. Also edit, delete, copy, paste, clear, and undo functionality. 
			PathDrawerMain.vi, the interface for creating auto paths. Will need to be updated for 2019.
	Dawgma Programming Library/
		Controls/, controls related things like PurePursuit and maybe the TurnToAngle via motion profiling
		RobotSimulator/, The robot simulator
		Utilities/, things like OneShotPulse.vi, TImerOnDelay.vi, Derivative.vi, BoxcarFilter.vi
		Vision/, Vision code from 2017, just to have it. 
		WPI Helper Stuff/, things that could sort of be in WPI library, so CheesyDrive.vi, CustomError.vi, GetPDPCurrents.vi.
