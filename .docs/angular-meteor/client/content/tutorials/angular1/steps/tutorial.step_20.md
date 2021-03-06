{{#template name="tutorial.step_20.html"}}
{{> downloadPreviousStep stepName="step_19"}}

In this step we are going to add the ability to upload images into our app, and also sorting and naming them.

Angular-Meteor provides us with the [$meteor.collectionFS API](/api/files) that wraps [CollectionFS](https://github.com/CollectionFS/Meteor-CollectionFS).

[CollectionFS](https://github.com/CollectionFS/Meteor-CollectionFS) is a suite of Meteor packages that together provide a complete file management solution including uploading, downloading, storage, synchronization, manipulation, and copying.

It supports several storage adapters for saving to the local filesystem, GridFS, or S3, and additional storage adapters can be created.

So let's add image upload to our app! We will start by adding CollectionFS to our project by running the following command:
```
meteor add cfs:standard-packages
```

Now, we will decide the the storage adapter we want to use. CollectionFS provides adapters for many popular storage methods, link FileSystem, GridFS, Amazon S3 and DropBox.
In this example, we will use the GridFS as storage adapters, so we will add the adapter adapter by running this command:
```
meteor add cfs:gridfs
```
Note: you can find more information about Stores and Storage Adapters on the CollectionFS's GitHub repository.

So now we have the CollectionFS support and the storage adapter - we need to create a CollectionFS object to handle our files.
Note that you will need to define the collection as shared resource because you will need to use the collection in both client and server side.

### Creating the CollectionFS

I create the `model/images.js` file, and define a regular CollectionFS object called "Images", also, I used the CollectionFS API that allows me to defined auth-rules.
Finally, I publish the collection just like any other collection, in order to allow the client to subscribe to those images:

{{> DiffBox tutorialName="angular-meteor" step="20.1"}}

And let's add the usage of $meteorCollectionFS to our core controller (`PartiesListCtrl`):

{{> DiffBox tutorialName="angular-meteor" step="20.2"}}

So now we have the collection, we need to create a client-side that handles the images upload.

### Image Upload

Note that you can use basic HTML `<input type="file">`  or any other package as you want - you only need the HTML5 File object to be provided.
For our application, we would like to add ability to drag-and-drop images, so we can use AngularJS directives that handles file upload and gives us more abilities such as drag & drop, file validation on the client side and some more.
In this example, I will use [ng-file-upload](https://github.com/danialfarid/ng-file-upload), which have many features for file upload.
In order to do this, lets add the package to our project:
```
meteor add danialfarid:ng-file-upload
```

Now, lets add a dependency in the `app.js` file:

{{> DiffBox tutorialName="angular-meteor" step="20.3"}}

Now, before adding the actual file upload to the client, let's add a `$mdDialog` that will show the file upload form, in the `partiesList.js` file:

{{> DiffBox tutorialName="angular-meteor" step="20.4"}}

Also, don't forget the $mdDialog dependency on the scope!

{{> DiffBox tutorialName="angular-meteor" step="20.4.1"}}

And add a button that calls our `openAddImageModal` function inside the create new party form:

{{> DiffBox tutorialName="angular-meteor" step="20.5"}}

Now let's create the view in a new file - `client/parties/views/add-photo-modal.ng.html` for this dialog and add the `ng-file-upload` directive:

{{> DiffBox tutorialName="angular-meteor" step="20.6"}}

Also, in order to make the "drop-zone" look like a dropable area in my page, I added this CSS in the `parties.import.less` stylesheet:

{{> DiffBox tutorialName="angular-meteor" step="20.7"}}

The main code in this HTML is the usage of `addImages` function in the `nfg-change` attribute, because this is the function that handles the main logic of uploading the image.
So let's create the controller (`addPhotoCtrl.js`) and implement this function:

{{> DiffBox tutorialName="angular-meteor" step="20.8"}}

And that's it! now we can upload images by using drag and drop!

Now let's add some more cool features!

### Image Crop

One of the most common actions we want to make with pictures is edit it before saving, we will add to our example ability to crop images before uploading them to the server.
In this example, we will use [ngImgCrop](https://github.com/alexk111/ngImgCrop/) which provides us the ability to do that.
So lets start by adding the package to our project:
```
meteor add alexk111:ng-img-crop
```
And add a dependency in our module:

{{> DiffBox tutorialName="angular-meteor" step="20.9"}}

We want to perform the crop in the client side only, before saving it in the CollectionFS, so lets get the uploaded image, and instead of saving it in the server - we will get the Data Url of it, and use it in the ngImgCrop:

{{> DiffBox tutorialName="angular-meteor" step="20.10"}}

We take the file object and used HTML5 FileReader API to read the file from the user on the client side, without uploading it to the server.
Then we saved the DataURI of the image into a $scope variable.
Next, we will need to use this DataURI with the ngImgCrop directive as follow:

{{> DiffBox tutorialName="angular-meteor" step="20.11"}}

And add `ng-hide` to the upload control, in order to hide after the user picks an image to crop.

{{> DiffBox tutorialName="angular-meteor" step="20.12"}}

And add some CSS to make it look better:

{{> DiffBox tutorialName="angular-meteor" step="20.12.1"}}

And let's change the logic of the "Done" button to call a function that will actually save the edited image:

{{> DiffBox tutorialName="angular-meteor" step="20.13"}}

In order to save, we need to implement `saveCroppedImage()` function. We will use the same `$meteorCollectionFS` API we used before, just use the `save` function.
CollectionFS have the ability to receive DataURI and save it, just like a File object. So the implementation looks like that:

{{> DiffBox tutorialName="angular-meteor" step="20.14"}}

We will save the image call the `answer` function that just close the modal and notifies the main page about the action the user made (save / cancel)

{{> DiffBox tutorialName="angular-meteor" step="20.15"}}

And because we can share the information from the recently closed dialog with the main view (the add party form), we can just the Promise of the `show` function and implement a logic that will save the list of images we want to attach to out party:

{{> DiffBox tutorialName="angular-meteor" step="20.16"}}

And we are done! Now we have the ability to crop images and then save them using CollectionFS.

### Display Uploaded Images

Let's add a simple gallery to list the images in the new party form:

{{> DiffBox tutorialName="angular-meteor" step="20.17"}}

Cool! We can the all the images we want to attach to the new party!

Now let's add description to our images!

### Images Metadata & Description

Using CollectionFS, we can also save metadata on the files we store.
In order to to that, we just need to update the image object with the value, on the `metadata` property of each image.

In order to do that really nice and user-friendly, I used [angular-xeditable](https://github.com/vitalets/angular-xeditable).
Lets add the angular-xeditable package to our project:
```
meteor add vitalets:angular-xeditable
```
And add a dependency in our module:

{{> DiffBox tutorialName="angular-meteor" step="20.18"}}

Now, let's use angular-xeditable and add usage under the image:

{{> DiffBox tutorialName="angular-meteor" step="20.19"}}

And, of course, implement `updateDescription` function on the parties list scope:

{{> DiffBox tutorialName="angular-meteor" step="20.20"}}

That's it! Now we have a photo gallery with description for each image!

### Sort Images

After we learned how to use CollectionFS metadata and how to update metadata values, we can also add an ability to sort and update the images order.
To get the ability to sort the images, let's use [angular-sortable-view](https://github.com/kamilkp/angular-sortable-view).

Add the package by running this command:
```
meteor add netanelgilad:angular-sortable-view
```
And then add a dependency for the module:

{{> DiffBox tutorialName="angular-meteor" step="20.21"}}

The basics of angular-sortable-view is to add the `sv-?` attributes to our page, just like the examples in the `angular-sortable-view` repository, So let's do that:

{{> DiffBox tutorialName="angular-meteor" step="20.22"}}

I also added `draggable="false"` to prevent the browser's default behavior for dragging images.

I use the `sv-on-sort` callback to know when the sort is done and which image moved to which position, the $partTo is the array with indexes updated after the sort is done.

Now let's implement the $scope function that will just save the order on the image as a local information, we will use it really soon.

{{> DiffBox tutorialName="angular-meteor" step="20.23"}}

We can also add a highlight to the first image, which we will use as the main image, by adding an indication for that:

{{> DiffBox tutorialName="angular-meteor" step="20.24"}}

And some CSS to make it look better:

{{> DiffBox tutorialName="angular-meteor" step="20.25"}}

So now we have image gallery with ability to edit and add images, add a description to the images, and ability to sort.
Now we just need to add the logic that connect those images with that party we are creating!

### Link Image To Object

So as you know, we have all the images stored in `newPartyImages` array, we let's implement the `createParty` function, and save a link to the image with it's order:
{{> DiffBox tutorialName="angular-meteor" step="20.26"}}

And now update the button to call this function:

{{> DiffBox tutorialName="angular-meteor" step="20.27"}}

And finally, to display the main image in the parties list, add the following code:

{{> DiffBox tutorialName="angular-meteor" step="20.28"}}

And implement the `getMainImage` function:

{{> DiffBox tutorialName="angular-meteor" step="20.29"}}

And that's it!

### Thumbnails
Another common usage with images upload, is the ability to save thumbnails of the image as soon as we upload it.
CollectionFS gives the ability to handle multiple Stores object, and perform manipulations before we save it in each store.
You can find the full information about image manipulation in the [CollectionFS docs](https://github.com/CollectionFS/Meteor-CollectionFS#image-manipulation).
Also, make sure you add `meteor add cfs:graphicsmagick` and install graphicsmagick, for more information, follow the instructions on the CollectionFS GitHub page.
So in order to add ability to save thumbnails, lets add a new store in the `model.js` and use `transformWrite` ability:

{{> DiffBox tutorialName="angular-meteor" step="20.30"}}

So now each image we upload will be saved also as thumbnail.
All we have to do is use and display the thumbnail instead of the original image. We just need to add a param to `url()` method of File objects and decide which Store to use when creating the URL:

{{> DiffBox tutorialName="angular-meteor" step="20.31"}}

That's it! Now we added the ability to save thumbnails for the images and use them in the view!

In addition to the angular-meteor-socially tutorial, you can find simple examples of each step of the tutorial in the Angular-Meteor repository, under examples/ directory.

{{/template}}
