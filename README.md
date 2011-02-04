# BETaskHelper

BETaskHelper helps you run NSTask interactively. Specifically you can:

- receive output of the task line-by-line
- send strings such as passwords to the task 

We use BETaskHelper in our apps [Archiver](http://archiverapp.com) and [MainMenu](http://incrediblebee.com/mainmenu).

## Example

This example uses the `openssl` command to encrypt a file.

In the header file implement the `BETaskHelperDelegate` protocol:

	#import "BETaskHelper.h"
	
	@interface FileEncrypter : NSObject <BETaskHelperDelegate> {
		BETaskHelper *taskHelper;
	}
	
	// Sets up and runs the find task (you can call the method anything you like)
	-(void) run;
	
	@end

Note that we keep the task helper as an instance variable because the task runs asynchronously.

In the implementation file use the `BETaskHelper` delegate methods to capture the output of the `openssl` command and send the password when requested:

	#import "FileEncrypter.h"
	
	@implementation FileEncrypter
	
	-(void) run {
		// Setup task
		NSTask *task = [[[NSTask alloc] init] autorelease];   // BETaskHelper will retain it for us
		[task setLaunchPath:@"/usr/bin/openssl"];
		NSString *inputPath = [NSHomeDirectory() stringByAppendingPathComponent:@"Report.txt"];
		NSString *outputPath = [NSHomeDirectory() stringByAppendingPathComponent:@"Report.enc"];
		
		[task setArguments:[NSArray arrayWithObjects:@"enc", @"-aes-256-cbc", @"-salt", @"-in", inputPath, @"-out", outputPath, nil]];
		
		// Hook up task with helper
		taskHelper = [[BETaskHelper alloc] initWithDelegate:self forTask:task];
		
		// Launch task -- must happen via task helper!
		[taskHelper launchTask];
	}
	
	#pragma mark BETaskHelper delegate methods
	
	-(void) task:(NSTask *)task hasOutputAvailable:(NSString *)outputLine {
		// Log all output
		NSLog(@"%@", outputLine);
	
		// Send password if requested (will be called twice for verification)
		if ([outputLine rangeOfString:@" password:"].location != NSNotFound) {
			// Most commands require newline character to signal end of input
			[taskHelper sendInput:@"YouWillNeverGuess\n"];
		}
	}
	
	-(void) task:(NSTask *)task hasCompletedWithStatus:(int) status {
		NSString *message = (status == 0) ? @"Success!" : @"Failed!";
		NSLog(@"%@", message);
	}

	@end

## License

Copyright (c) 2007, Incredible Bee Ltd. All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

- Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
- Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
- Neither the name of the Incredible Bee Ltd. nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL INCREDIBLE BEE LTD. BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.