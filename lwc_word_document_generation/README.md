Video tutorial for how to generate word documents from Lightning Web Components: https://youtu.be/VU0SnB5dC-g


Salesforce Development Tutorials | Salesforce Development: Lightning Web Components (LWC)
Salesforce Development Tutorial(LWC): How to Generate a Word Document from a Lightning Web Component
By
Coding With The Force Inc
January 20, 2021


[sm-youtube-subscribe]
Why Create an LWC that can Generate Word Documents?

This is a little more self explanatory than many of the blog posts I do, but let’s go over some things. You typically wanna create this because the business has a need (for one reason or another) to generate a word doc. I’ve had businesses need them so important people could sign off on something with a hand written signature, needed guest list printed for campaigns/events and several other scenarios.

As far as why we should use an LWC to do this instead of a VF Page or Aura Component, Aura Components are considerably slower and I would just suggest not making them anymore in general and VF Pages suffer from view state limitations. While it’s easier to deal with them when working with external libraries because of lightning locker service, it’s easy to generate a document with images that blows past the 170kb view state limit and then crashes your page.

The docx.js Javascript Library

To generate word documents, we need to use the docx.js javascript library, which thankfully, is locker service compliant! Saves us a lot of time (if you didn’t know you can modify most libraries to make them compliant). You can get the docxjs code we’re gonna be using for this tutorial here .

This library basically allows you to generate word documents using javascript. It makes your life doing this a thousand times easier, so make sure to thank the devs that built it!

Writing the Code

The code we’re gonna write to get this done is just for a simple example. We’re gonna generate a list of contacts associated with an account in a word document. Before we get started, all this code is up on my GitHub here, so if you wanna just ignore this whole section and check out the GitHub repo, feel free, otherwise, please carry on, lol. So first things first, open up VSCode and create a new lightning web component! If you aren’t familiar with how to setup VSCode, I have a video covering it here!

Once you’ve got your new LWC created in VSCode, we need to upload the docxjs code to static resources so that we can use it in our LWC. You can grab the docxjs code here. Then navigate to static resources in setup and upload the docxjs code there. Make sure to make the static resource public!

After that’s done, switch back over to VSCode and let’s import the docxjs file into the LWC by using the code below:

import { LightningElement} from 'lwc';
import {loadScript} from "lightning/platformResourceLoader";
import docxImport from "@salesforce/resourceUrl/docx";

export default class Contact_list_generator extends LightningElement {

    connectedCallback(){
        Promise.all([loadScript(this, docxImport)]).then(() =>{
            //call some code here
        });
    }
}
You may be looking at the above like, “wtf is that bro?” so let me explain. The connectedCallback method is called when your LWC is loaded into the browser, so it’s kinda like the init method in Aura components. Promise.all is just saying, “Hey, I promise to wait until all the scripts are loaded, then I’m gonna execute the code inside this code block”. The loadScript is a module that Salesforce provides to you that allows you to load in resources to your LWC from static resources. Last, but certainly not least, the, “import docxImport from “@salesforce/resourceUrl/docx”;” is the actual reference to your docx static resource file. The docx at the end of the of that line should be whatever you actually named your static resource.

Next let’s add the html below to our LWC

<template>
    <div class="slds-p-bottom_x-large">
        <lightning-button class="hidden slds-float_left slds-p-right_medium" onclick={startDocumentGeneration} label="Build Document"></lightning-button>
        <a href={downloadURL} download="ContactList.docx" class="slds-hide slds-button slds-button_brand slds-float_left" >Download Document</a>         
    </div>
</template>
In the HTML above we’re basically just creating two buttons, one to generate a word document and one to download that word document. You’ll notice there are two references in the HTML to js variables/methods that don’t exist yet (startDocumentGeneration and downloadURL) so let’s get back to the LWC js controller and figure this thing out.

The next thing we need to add is a way to render the generate document button after the component loads the docxjs script. We can do that with the following code

    connectedCallback(){
        Promise.all([loadScript(this, docxImport)]).then(() =>{
            this.renderButtons();
        });
    }

    renderButtons(){
        this.template.querySelector(".hidden").classList.remove("hidden");
    }
In our connected callback method we’re gonna call a method called renderButtons that changes the visibility of our buttons after our scripts are loaded. The “this.template.querySelector(“.hidden”).classList.remove(“hidden”);” is removing the class that was hiding the component and allowing it to be viewed and clickable. We do need to actually add the css though. So let’s add the css below to the component

.hidden{
    display: none;
}
Basically that css just allows you to hide an element… pretty simple. Not much there.

The next thing we need to do is create the startDocumentGeneration method and actually grab our contact data and build the document. So let’s look at the rest of the controller code we need to build out below.

import { LightningElement, api } from 'lwc';
import {loadScript} from "lightning/platformResourceLoader";
import docxImport from "@salesforce/resourceUrl/docx";
import contactGrab from "@salesforce/apex/ContactGrabber.getAllRelatedContacts";

export default class Contact_list_generator extends LightningElement {

    @api recordId;
    downloadURL;
    _no_border = {top: {style: "none", size: 0, color: "FFFFFF"},
	bottom: {style: "none", size: 0, color: "FFFFFF"},
	left: {style: "none", size: 0, color: "FFFFFF"},
	right: {style: "none", size: 0, color: "FFFFFF"}};

    connectedCallback(){
        Promise.all([loadScript(this, docxImport)]).then(() =>{
            this.renderButtons();
        });
    }

    renderButtons(){
        //this.template.querySelector(".hidden").classList.add("not_hidden");
        this.template.querySelector(".hidden").classList.remove("hidden");
    }

    startDocumentGeneration(){
        contactGrab({'acctId': this.recordId}).then(contacts=>{
            this.buildDocument(contacts);
        });
    }

    buildDocument(contactsPassed){
        let document = new docx.Document();
        let tableCells = [];
        tableCells.push(this.generateHeaderRow());

        contactsPassed.forEach(contact => {
            tableCells.push(this.generateRow(contact));
        });

        this.generateTable(document, tableCells);
        this.generateDownloadLink(document);
    }

    generateHeaderRow(){
        let tableHeaderRow = new docx.TableRow({
            children:[
                new docx.TableCell({
                    children: [new docx.Paragraph("First Name")],
                    borders: this._no_border
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph("Last Name")],
                    borders: this._no_border
                }) 
            ]
        });

        return tableHeaderRow;
    }

    generateRow(contactPassed){
        let tableRow = new docx.TableRow({
            children: [
                new docx.TableCell({
                    children: [new docx.Paragraph({children: [this.generateTextRun(contactPassed["FirstName"].toString())]})],
                    borders: this._no_border
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph({children: [this.generateTextRun(contactPassed["LastName"].toString())]})],
                    borders: this._no_border
                })
            ]
        });

        return tableRow;
    }

    generateTextRun(cellString){
        let textRun = new docx.TextRun({text: cellString, bold: true, size: 48, font: "Calibri"});
        return textRun;
    }

    generateTable(documentPassed, tableCellsPassed){
        let docTable = new docx.Table({
            rows: tableCellsPassed
        });

        documentPassed.addSection({
            children: [docTable]
        });
    }

    generateDownloadLink(documentPassed){
        docx.Packer.toBase64String(documentPassed).then(textBlob =>{
            this.downloadURL = 'data:application/vnd.openxmlformats-officedocument.wordprocessingml.document;base64,' + textBlob;
            this.template.querySelector(".slds-hide").classList.remove("slds-hide");
        });
    }
}
So, there’s a bit to cover here, lol, so let’s start with the call in to the apex controller to get our contacts. This line of code here:

contactGrab({'acctId': this.recordId}).then(contacts=>{
            this.buildDocument(contacts);
        });
This calls to the apex controller and retrieves a list of contacts based on the account id of the record we’re currently on. If you didn’t know, the @api recordId variable declaration at the top of the class just dynamically pulls in the id of the record your component is being viewed on! Super convenient! We are also able to call our apex method using the contactGrab({‘acctId’: this.recordId}) statement because we imported our apex class at the top of the LWC here “import contactGrab from “@salesforce/apex/ContactGrabber.getAllRelatedContacts”;”. That being said we haven’t looked at the apex code yet, so let’s check it out… there’s not much there but it’s still important.

public with sharing class ContactGrabber {
    @AuraEnabled
    public static List<Contact> getAllRelatedContacts(Id acctId){
        return [SELECT Id, FirstName, LastName FROM Contact WHERE AccountId = :acctId];
    }
}
The @AuraEnabled declaration allows us to import this method in our class to the LWC. It’s import to do that, so don’t forget!

Now that we have our contacts we actually need to generate the document. So let’s get to it bruh! The buildDocument method starts this process so let’s check it our first.

buildDocument(contactsPassed){
        let document = new docx.Document();
        let tableRows = [];
        tableRows.push(this.generateHeaderRow());

        contactsPassed.forEach(contact => {
            tableRows.push(this.generateRow(contact));
        });

        this.generateTable(document, tableRows);
        this.generateDownloadLink(document);
    }
In the code above we’re declaring a new docx Document object with the “new docx.Dcoument()” declaration. After this we create an array of table cells (because in this example we are building a table of contacts in a word document). We then proceed to push a table row into the table rows array by calling the generateHeaderRow method in our js controller. Let’s check out that class next.

generateHeaderRow(){
        let tableHeaderRow = new docx.TableRow({
            children:[
                new docx.TableCell({
                    children: [new docx.Paragraph("First Name")],
                    borders: this._no_border
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph("Last Name")],
                    borders: this._no_border
                }) 
            ]
        });

        return tableHeaderRow;
    }
The generateHeaderRow method use the docx.TableRow object, the docx.TableCell object and the docx.Paragraph object to generate a table row with two cells. One cell for the contacts first name and another cell for a contacts last name. It then returns this table row.

Let’s get back to the buildDocument method now. The next thing that happens is we iterate through the list of contacts that we pulled from our apex controller and generate a table row for each contact by calling the generateRow method and push that into our tableRows array. So let’s look at the generateRow method next.

generateRow(contactPassed){
        let tableRow = new docx.TableRow({
            children: [
                new docx.TableCell({
                    children: [new docx.Paragraph({children: [this.generateTextRun(contactPassed["FirstName"].toString())]})],
                    borders: this._no_border
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph({children: [this.generateTextRun(contactPassed["LastName"].toString())]})],
                    borders: this._no_border
                })
            ]
        });

        return tableRow;
    }
This code does something similar to the generateHeaderRow method, the only difference between the two is that I call the generateTextRun method instead of just outright declaring a new docx Paragraph object. The docx.TextRun object allows us to specify traits in our text. Things like font size, font type, whether the text is bold and a ton more. Let’s check out the generateTextRun method to see what it’s doing.

generateTextRun(cellString){
        let textRun = new docx.TextRun({text: cellString, bold: true, size: 48, font: "Calibri"});
        return textRun;
    }
In the method above we are generating what docxjs calls a text run and then returning it. It’s pretty simple as you can see. I’m just declaring the traits I want for my text. Nothing more, nothing less.

Back to the buildDocument method then! The next thing we do is call the generateTable method and pass is our docx.Document object along with our array of tableRows. Let’s check out that method next!

generateTable(documentPassed, tableCellsPassed){
        let docTable = new docx.Table({
            rows: tableCellsPassed
        });

        documentPassed.addSection({
            children: [docTable]
        });
    }
In this method we are creating a new docx.Table and assigning the array of rows we passed to this method to the rows parameter of the docx.Table. We then proceed to add a new section to our docx.Document and put the table in that section. This actually adds the table to the document we are creating.

Now, one last time, let’s check out the buildDocument method again. The last thing we do in it is call the generateDownloadLink method. So let’s take a look at that method now.

generateDownloadLink(documentPassed){
        docx.Packer.toBase64String(documentPassed).then(textBlob =>{
            this.downloadURL = 'data:application/vnd.openxmlformats-officedocument.wordprocessingml.document;base64,' + textBlob;
            this.template.querySelector(".slds-hide").classList.remove("slds-hide");
        });
    }
What this method does, is take the document we built and create a url that will allow us to download the word doc. It also turns on the download button in our LWC. We generate a base64 encoded string using the docx.Packer object and assign it to the downloadURL.

And believe it or not, that’s it! Yea!!! You’ve just figured out how to build your own LWC that can produce word documents. You can build off this base to do whatever you think you might wanna do. You could, with the help of other js libraries build a whole document templating app. I’ve done it in the past, it’s challenging, but doable! Good luck building whatever it is you’re building with this!

Get Coding With The Force Merch!!

We now have a redbubble store setup so you can buy cool Coding With The Force merchandise! Please check it out! Every purchase goes to supporting the blog and YouTube channel.

Get Shirts Here!
Get Cups, Artwork, Coffee Cups, Bags, Masks and more here!

Check Out More Coding With The Force Stuff!

If you liked this post make sure to follow us on all our social media outlets to stay as up to date as possible with everything!

Youtube
Patreon
Github
Facebook
Twitter
Instagram

Salesforce Development Books I Recommend

Advanced Apex Programming
Salesforce Lightning Platform Enterprise Architecture
Mastering Salesforce DevOps

Good Non-SF Specific Development Books:
