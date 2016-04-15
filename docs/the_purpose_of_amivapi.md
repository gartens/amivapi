# The Purpose of *amivapi*

*amviapi* was created to renew the IT infrastructure of the student organisation *AMIV*, but we believe it might be helpful to similar organisations as well. The main goal is to create a RESTful API to serve as a backend for our needs.
We are trying to keep the functions to a relevant minimum.

This document aims to show what the API is supposed to do. It is supposed to show the usecases and functionality, but **not** the specific implementation, because it is intended to be use as a reference for said implementation. If you want to see how it works, take a look at the source, the best starting points are *bootstrap.py*, *models.py* and *schemas.py*

## Who is using the API?

Different kinds of users will be able to interact with the API. They will be introduced here (and referred to in other parts of this document)

* **Public users**

  This user has basically just opened the amiv website and has not logged in. This might be someone from ETH, a highschool student checking out amiv or members that haven't logged in yet.
  
* **Registered users**

  This user is in our user database and has successfully logged in. This does *not* mean that he/she is a member of our organisation.
  
TODO: Should a non member be able to log in or not?
  
* **Privileged Users**

  As member above, but this member has a function in the organisation which requires more permissions.This could be a board member, event organisator, an API admin, etc. .
  Depending on the resource there might be different groups of privileged members.
  This doesn't have to be tied to a team in the organisation, e.g. a user has more privilege trying to access his own data compared to trying to access others.


## Resources

With the users above in mind we will now take a look at the resources the API provides, describe their purpose and how different users will be able to interact with them.


### Users ( */users* )

This are all registered users. We save the following data:
* **nethz** for login with ETH servers
* **email** for main communication, also as login if nethz is not given (or no longer available)
* **password** if login via ETH server is not wanted, the API can perform a login as well and needs a password for this.
* **membership** if someone is a member, extraordinary member or honorary member.
* **full name** for identification at events etc.
* **gender** mostly because there are some *female only* events by amiv team *limes*
* **phone number** (optional), mostly for active members so they can be reached easily.
* **legi number and RFID** to be able to connect a legi scan to a member. The legi number should be hashed for security reasons. It is important to find a user by given legi, not the other way round.

Furthermore we need the information if the user wants to receive our newsletter and we want to keep reference to a users groups, sessions and signups (see below)

User data is synchronized with ETH LDAP (but it can not be guaranteed how fast changes will propagate).

#### Public users
Public users can't see any user data.

#### Registered users
Everybody who can log in is able to view his/her own data and add/change a password, change mail/phone and adjust newsletter settings. Also the RFID needs to be changeable in case someone gets a new legi.
Membership can not be changed.
It is also not possible to change personal data like name or gender without an admin to prevent posing as a different person. 
Finally data provided by ETH (legi number, nethz, department) can not be changed eitehr

But it is still not possible to see any other user besides themselves

#### Privileged users
Some users need administration privilege to find all users and edit all data. Even data provided by ETH must be editable in case something was imported incorrectly.

---
### Sessions ( */sessions* )
User login sessions.

#### Public user
Every public user must be able to attempt to log in. This is possible either with eth user/pass or email and password as specified in the API database.
Public user can't see any sessions

#### Registered users
All own sessions can be seen and deleted.

#### Privileged users
Some administration privilege is needed to be able to check and remove sessions, but it is not possible to create a session for someone else.

---

### Groups ( */groups* )

A group is a team within the organization. E.g. the "Vorstand" (board) or "Kulturteam" (event team) would be groups. Therefore all a group needs is a unique name.
Groups are also mailing lists. Several group addresses ( */groupaddresses* ) can be spezified. 
Any mail sent to any of those addresses will be forwarded to all group members ( */groupmembers* ) as well as optional external forward addresses ( */groupforwards* ).

A group has a **moderator** that can add and remove members, addresses and forwards.
It is possible to mark a group as **public** to open it for self-registration.

TODO: Differentiate between public for ETH and public for members? i. e. allow non members to join groups?

Groups are often tied to certain tasks, e.g. the event team needs to be able to manage events. Therefore it is possible to add **permissions** to a group, i.e. give all group members admin rights for certain API functionality. This happends on endpoint/method level, e.g. a group could grant admin rights for POST and PATCH to /users (including /users/id)

Also the amiv data server *Zoidberg* can create storage directories for groups. For this purpose it can be specified if storage is needed or not.

#### Public users
No groups are visible or accessible.

TODO: Should someone be able to check if his mail is in a group and be able to remove it?

#### Registered users
All **public** groups are visible as well as groups of which the user is a member. Other members in those groups can also be seen (to see who is on your team)
Registered users can join any **public** group and leave all groups they are member of. They cannot modify the group itself.

#### Priviledged users
* The **group moderator** can add and remove users and addresses of his group. He can change everything except the group permissions (since his could be used for full API privilege)
* General group administrators can do all of the above but also modify permissions. This status should be given with care.

---

### Events ( */events* )

An event is anything happening within AMIV you want others to know of and join. For example, the "Kulturteam" (event team) may announce a barbacue with beer and sausage.

Events may offer the following information:

* **time and location**
* **price** This information is seperated from the description as it is an important information that should be presented to everybody before signing up. Furthermore, this way we offer the possibility for electronic payments.
* **spots** and **registration timeframe** Many events are limited concerning the number of people attending. Therefore, they require registration in advance ( */eventsignups* ). To be fair to all users, the event may be published before the opening of the registration so people have equal chances to register as soon as the timeframe opens. Furthermore, people can sign up for an event no matter if there are already more registrations than spots. They are kept in a queue in this case.
* **registration form** In some cases, the event organizer requires additional information in advance, e.g. whether attendees have a "Halbtax" (train discount card) or if they are older than 18 years.
* **description** and **images** mostly to promote events to members in an attractive way. As many MSc students do not speak German, organizers may promote an event in English aswell.
* **promotion settings** to steer the promotion of an event, organizers can set in which contexts the event is promoted (in the newsletter / on the homepage / ...)


#### Public users
Can see events and can register to events that are open to non-members. As we do not have any contact information for them, they need to register with an email address and confirm this address.

#### Registered users
Can see events and register to all events as soon as they are logged in.

#### Priviledged users
* The **event owner** can see signups for his/her events and also change and delete registrations and the complete event.
* General event administrators can do all the above but for all events.



TODO: Rest of resources
