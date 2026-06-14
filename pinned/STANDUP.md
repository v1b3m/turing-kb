Mar 5:  

*Work Done*  

- Initial batch of communication feature (emails)

Time: 4h

*Plan For Tommorrow*

- Complete communication feature

*Blocker*  

- Emails are sent out in a popup window which does not share storage (local + session) with the main window. This means using storage to share context is not going to work. We can close this gap with an API (any window from anywhere can just call the API directly to update content).
- Currently using broadcast channel to sync content.
-  

