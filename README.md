# Letters/Emails from LegalServer
This package allows for the user to generate a letter or email in docassemble for a case from LegalServer.

## Requirements
This requires the `Docassemble-GBLS` package. Please follow the instructions in that package to connect your LegalServer site with your docassemble instance. You will need a separate API key for each site (LS live vs. LS demo). From that package, you may also want to customize the `LS_fields` command to pull the appropriate fields necessary in your system. See the Customize section below for more details. The default here pulls `client`, `advocate`, `pbadvocate`, and `initiator` as separate individuals. The `Pro Bono Advocate/Volunteer` is `pbadvocate`, the `Primary Assignment` is the `advocate`, and the `User` in LegalServer is the `initiator`. This is to allow for letters being sent by a paralegal assisting the advocate instead of just the advocate. 

This project requires the Docassemble configuration: `new markdown to docx: True` to properly generate the letters on the letterhead in the appropriate format. Adding the `convertapi` feature to Docassemble may be necessary depending on the graphics of your letterhead and how `pandoc` handles the PDF conversion of that letterhead. It may also be helpful to add the `use cloud urls: False` configuration item if there are problems downloading the PDF/DOCX versions of letters.

This requires Docassemble 0.5.99 given the presence of the DACatchall features. 

## Functionality
The user will have the option of selecting the recipient (either the Client or the Pro Bono Volunteer (if there is one)), then the sender (either the LegalServer user who started the process or the LegalServer Advocate on the case), and then the message they wish to send. The message can then be editted if there are any changes to the standard text before sending. 

Emails are sent via the `send_email` function in Docassemble. They are sent to the message recipient while copying the `case_email` from LegalServer. The template subject is the subject for the email. The user is copied on these emails. Before the email is finally sent, there is the option to add attachments to the email message. 

Letters can be sent to the Client or the Pro Bono Volunteer using either email or letters. The final version is sent back to the LegalServer `case_email` and saved in the case notes of the case as both a DOCX and a PDF. The user is then given download links to open and print the letter. Please recognize that if you are using a combination of full page letters and envelopes (as in the example), it cannot be properly printed from within the browser's print feature (everything will just print on letter size paper). You have to use your dedicated PDF reader or the DOCX file to properly print.  

There are a number of custom variables that this pulls from LegalServer. `legal_problem_code_num` is the integer value of the first two characters of the `Legal Problem Code`. This may generate an error if those are not numeric. `special_legal_problem_code_num` is the integer value of the first three or four charachters of `Special Legal Problem Code` if that was available. `LPC` is the name of the `Special Legal Problem Code` if that is available and of the `Legal Problem Code` if the `Special Legal Problem Code` is not available. This is used in the subject of the letters/emails in the example. `casenumber` is similarly used, but a little more obvious.  

In sending letters, there is a new feature to send to PostalMethods. This is an automated letter printing service. By sending a letter to them as an attachment, they will print it and mail it for you if you have an existing account with a postive balance and are using the template for their letters. It requires setting up the template for a two window envelope. 

## Customization
### Sample
Take a look at the `LSLetters-sample.yml` file that is in this package. This is an example file that you could install as a separate docassemble package. This has all the basic aspects of the letter that you can then modify as you need with comments included on each docassemble block. 

### Generally
It is expected that in using this package, you will have installed this package, `docassemble-GBLS`, and your custom package. Your package should include your metadata to identify it, a line to include this package: `include: - docassemble.LSLettersEmails:LSLetters.yml` (with a return after the include line), and any customized questions, letter text, and letterhead templates to replace what's in the standard package. You can also capture additional (non-default) fields from LegalServer, ask additional questions in Docassemble and set defaults based on existing fields. You will then need to add your package to the `dispatch` configuration option to have it properly display in LegalServer. There is no need to add this package to your `dispatch` configuration.  

### Letterhead details
The letterhead template files are saved in the `templates` folder with one for the client and one for the volunteer attorney. You will also need to use your own letterhead. Basic default letterhead should be saved as `letterhead-pbadvocate.docx` and `letterhead-client.docx`. This should override the HVL versions that are installed as a default into the package. Please note that you can include an envelope with the letterhead to plan for those as well. New letterhead files should also be included as additional attachments and in the Letterhead question. See `LSLetters-customize.yml` for examples of how this all works. Keep a default option in the `Letterhead` question to default to the standard letterhead options. If you do not want your package to prompt for the letterhead options, you can include an `initial` block that sets `clientmessage_letterhead = "default"`. Additional versions of letterhead beyond the two defaults in the system will require their own `attachment` block in the same format as the two in the example. 

If you are going to be using multiple versions of this package with a standardized set of letterhead/templates, it may be best to fork the current version of this package, revise the templates, attachment blocks, and `clientmesage_letterhead` code, and install that package on your server. Then you can have that information centralized while the letter templates are smaller, one YAML file packages with the different templates required for different groups, purposes, or subjects. 

### Letter Texts
The actual text of the letters is contained as `template` blocks in the `LSLetters-customize.yml` file included in this package. The name of the template should be included as a choice in the first block in that document -- `clientmessage.textoption`. Letters should start with the `toline` and end with the `fromline`. There are custom variables for these fields -- `clientmessage.toline` and `clientmessage.fromline` that are based on who the sender/receipient are, their gender, their language, and salutation. Plan on overwriting the `clientmessage.textoption` and the appropriate templates with your organizations content by adding these fields to a new YML file that includes: `docassemble.LSLettersEmails`. When creating the `client.toline`, it will use `Dear Mr. Smith` as the toline if there is a language and a gender. If the gender is not `male` or `female`, it will use the client's entire name -- `Dear James Smith` -- instead. This is done by the `greeting` language option in the Python module associated  with the package.

### Non-English Language Options
It is presumed that the language included is English for anyone other than the client. For non-english letters, please see the example. Internal docassemble translation tools are not use to keep the interview itself in English. This includes languages for the client in English, Spanish, Chinese, and Vietnamese. It does not do additional languages. For that you will need to:
* edit `docassemble.GBLS` to recognize other language codes based on language names from LegalServer
* add a code block that defines the `client.solicitation_custom` and the `client.title` fields for that language. Note that there are two client.title fields -- one final field `client.title` and one for each language. 

### Additional Fields/Variables
Capturing additional fields from LegalServer is completely possible. The Profile or Auxilliary Process that has the Docassemble instruction described in the `docassemble-GBLS` package will allow you to capture any/all of the visible fields. You will need to test this to make sure that the field is capturing data in a usable format. To use this to automatically set certain variables, define it in an `initial` block in your code. It should read like `variable = ls_fields['LSFieldName']` where `variable` is the Docassemble variable you are assigning to it and `LSFieldName` is the field in LegalServer you are attempting to capture. Test to make sure that it is working and review the variable log to see that the names are correct. Capitalization matters, so it may require multiple tests. Also, build in error handling for when that field is blank or offers invalid data. 

Alternatively, this interview uses the (DACatchall features)[https://docassemble.org/docs/fields.html#catchall], so if an undefined variable is asked, it will prompt with a question. Please make sure any new variables used have sufficiently identifiable names so that the catchall features are clear. Note that there are certain terms in the variable name that have custom results -- `signature` will call a signature block, variables including the term `time` will call a time datatype, and variables including `date` will have a date datatype. 

### Default options
Defaults to the questions asked here can also be set in the same `initial` block. For example, if you want to default the client language to Spanish, you can include the line `client.language = "es"` in the `initial` block and it will overwrite what is coming in from LegalServer. Alternatively, you can default one of the variables used in this interview. For example, if you said `clientmessage.recipient = client` it would automatically chose the client as the recipient of the messages for any template you select. This can be useful when grouping and organizing form letters. 
  
### BCC Case Email
The system also gives you the option on how to send the information to LegalSever. For letters that are sent by email, you will get the question of whether the case email at LegalServer should be on the cc line (`false`) or bcc line (`true`). Which option you want is probably an organizational decision. You can make this a default by including `bcc_case_email = true` in the `initial` block. There is a sample of this. 

### Mailgun/Email Notifications
If you are using Mailgun, you will not automatically get bounce and failure notices regarding emails that did not properly send. You can use Zapier or another service to configure the webhooks in Mailgun to report those bounces to a different authority. For example, HVL configured those to report to a MS Teams Channel. 

### Change Log

See the separate file for the Change Log. 