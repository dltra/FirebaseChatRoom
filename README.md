# FirebaseChatRoom
## Setting up Firebase Database
* In Chrome:  Log into your Firebase account
* In Android Studio:  click **Tools**
  * **Firebase**
	* **Realtime Database**
	* Save and retrieve data
	* Connect your app to Firebase
	  *	You can set the Firebase project name at this time
	* Add the Realtime Database to app
		* Accept the gradle changes
* In Chrome Firebase console:  In your Firebase account
	* Select your newly created project
	* Click Database
	* Scroll down to Realtime Database
		* Create Database
		* Start in Test mode to allow read and write permissions

* In Android Studio:  Modify MainActivity.java
```	
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    // Write a message to the database
    FirebaseDatabase database = FirebaseDatabase.getInstance();
    DatabaseReference myRef = database.getReference("message");
    
    myRef.setValue("Hello, World!");
}
```
    
* import packages
```
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
```
* Test App
---
* In Android Studio
  * Add an event listener to your database reference inside of OnCreate
```
// Read from the database
myRef.addValueEventListener(new ValueEventListener() {
    @Override
    public void onDataChange(DataSnapshot dataSnapshot) {
        // This method is called once with the initial value and again
        // whenever data at this location is updated.
        String value = dataSnapshot.getValue(String.class);
        System.out.println("Firebase test: Value is: " + value);
    }
    @Override
    public void onCancelled(@NonNull DatabaseError databaseError) {
        // Failed to read value
        System.out.println("Failed to read value."+ databaseError.toException());
    }
});
```
* Test by modifying the value in Chrome and observing the Logcat
---
* In Chrome Firebase console: 
	* Click on Authentication
		* Sign-in method
			* Email/Password
      * Enable
			* Save
  * In Android Studio
    * Firebase => **Authentication** => Add Firebase Authentication to your app => Accept Changes (to build.gradle)      
      * Sync gradle
    * In MainActivity.java
      * Add a global FirebaseAuth mAuth reference
      * Instantiate the reference in onCreate
      * In onStart check if user is signed in and update accordingly
```
@Override
public void onStart() {
    super.onStart();
    // Check if user is signed in (non-null) and update UI accordingly.
    FirebaseUser currentUser = mAuth.getCurrentUser();
    updateUI(currentUser);
}
private void updateUI(FirebaseUser user) {
    if (user != null) {
        System.out.println("Authen Test:  Email: "+user.getEmail()+ "; Status: "+ 
                user.isEmailVerified() +"; ID: "+ user.getUid());
    } else {
        System.out.println("Authen Test:  No verified user");
    }
}
```
* Test: Logcat should show "No verified user"
* In Android Studio
  * In onCreate
    * Create a dummy user with email and 6 character password
```
mAuth.createUserWithEmailAndPassword("abc@abc.com", "tjhsst")
        .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
            @Override
            public void onComplete(@NonNull Task<AuthResult> task) {
                if (task.isSuccessful()) {
                    // Sign in success, update UI with the signed-in user's information
                    System.out.println("Authen test:  createUserWithEmail:success");
                    FirebaseUser user = mAuth.getCurrentUser();
                    updateUI(user);
                } else {
                    // If sign in fails, display a message to the user.
                    System.out.println("Authen test:  createUserWithEmail:failure"+task.getException());
                    Toast.makeText(getApplicationContext(), "Authentication failed.",
                            Toast.LENGTH_SHORT).show();
                    updateUI(null);
                }
            }
        });
```
* Test:  You should see the new user in Logcat and on the Firebase console under Authentication/Users tab
---    
* In Android Studio
  * In onCreate
    * Sign into the dummy user with email and 6 character password
```
mAuth.signInWithEmailAndPassword("abc@abc.com", "tjhsst")
   .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
       @Override
       public void onComplete(@NonNull Task<AuthResult> task) {
           if (task.isSuccessful()) {
               // Sign in success, update UI with the signed-in user's information
               System.out.println("Authen test:  signInWithEmail:success");
               FirebaseUser user = mAuth.getCurrentUser();
               updateUI(user);
           } else {
               // If sign in fails, display a message to the user.
               System.out.println("Authen test:  signInWithEmail:failure"+task.getException());
               Toast.makeText(getApplicationContext(), "Authentication failed.",
                       Toast.LENGTH_SHORT).show();
               updateUI(null);
           }
       }
   });
```
* Test
---
* In Chrome Firebase console
	* Click on Database
		* Create Firestore 
		* Start in test mode ``` allow read, write: if request.auth.uid !=null; ```
		* Next and Done

* Firebase => **Firestore** => Add Cloud Firestore to your app => Accept Changes (to build.gradle)
 *    implementation 'com.google.firebase:firebase-storage:19.1.0'
 *    implementation 'com.google.firebase:firebase-firestore:21.3.1'
---
* In Chrome Firebase console
	* Click on Storage
    * Get Started
    * Accept default security rules

* In Android Studio    
    * Firebase => **Storage** => Add Cloud Storage to your app => Accept Changes (to build.gradle)
    * Declare a global StorageReference mStorageRef and instantiate it in onCreate
    * Upload a file
```
        mStorageRef = FirebaseStorage.getInstance().getReference();
        Uri file = Uri.fromFile(new File("///storage/emulated/0/Download/download.jpeg"));
        StorageReference treeRef = mStorageRef.child("images/"+file.getLastPathSegment());
        treeRef.putFile(file)
                .addOnSuccessListener(new OnSuccessListener<UploadTask.TaskSnapshot>() {
                    @Override
                    public void onSuccess(UploadTask.TaskSnapshot taskSnapshot) {
                        // Get a URL to the uploaded content
//                        Uri downloadUrl = taskSnapshot.getDownloadUrl();
                        Task<Uri> urlTask = taskSnapshot.getStorage().getDownloadUrl();
                        while (!urlTask.isSuccessful());
                        Uri downloadUrl = urlTask.getResult();
                        System.out.println("Storage Test: "+ downloadUrl);
                    }
                })
                .addOnFailureListener(new OnFailureListener() {
                    @Override
                    public void onFailure(@NonNull Exception exception) {
                        // Handle unsuccessful uploads
                    }
                });
```
