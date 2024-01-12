<p><a target="_blank" href="https://app.eraser.io/workspace/18LcLCCLXKxtpvQ6rrf4" id="edit-in-eraser-github-link"><img alt="Edit in Eraser" src="https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fgithub%2FOpen%20in%20Eraser.svg?alt=media&amp;token=968381c8-a7e7-472a-8ed6-4a6626da5501"></a></p>

# Car Wash
![Figma](https://img.shields.io/badge/figma-%23F24E1E.svg?style=for-the-badge&logo=figma&logoColor=white "")

![License](https://img.shields.io/static/v1?label=license&message=CC-BY-NC-ND-4.0&color=green "")

## Table of Contents
[﻿Roadmap](https://github.com/Keith3895/car-wash/blob/main/README.md) 

### Introduction
This is a car wash application that allows users to book car wash services from the comfort of their homes. The application will be available on both Android and iOS. The application will be written in Flutter and Django.

### Assets
- UI Screens: [﻿link to Figma](https://www.figma.com/file/QbtwZGMvLuDpGXR2IcB7A3/Car-wash?type=design&node-id=0%3A1&mode=design&t=n68WTHlcaHpY86EO-1)  
- Business model canvas: [﻿Link to BMC](https://app.mural.co/t/keithfranklin6506/m/keithfranklin6506/1699966821684/e68a321ade3318162db526e676f4d3042aea0be4?sender=ud754661bd71ca9e639dd5444) 
### DevOps Design
#### Code Repository Design
The Application is split into two broad repositories:

1. Car Wash backend [﻿github.com/Keith3895/car-wash-backend-core](https://github.com/Keith3895/car-wash-backend-core) 
This is a Python app written with Django.
2. Car wash App [﻿github.com/Keith3895/car-wash-app](https://github.com/Keith3895/car-wash-app) 
This is a Flutter app created for both Android and iOS.
[﻿repo Design](https://app.eraser.io/workspace/18LcLCCLXKxtpvQ6rrf4?elements=34cwCizl3yFU2Wua60RqHA)  
### Application Design
#### DB Design
[﻿DB Design](https://app.eraser.io/workspace/18LcLCCLXKxtpvQ6rrf4?elements=xA0tEslogVhQjNqXsEwxyQ)  

#### Application Architecture 
##### FrontEnd:
The front end will be in a Flutter app. The source code structure will be:

```
car_wash
├─ .metadata
├─ .vscode
│  └─ settings.json
├─ README.md
├─ analysis_options.yaml
├─ assets                      # Contains all the assets of the project
├─ devtools_options.yaml   
├─ lib
│  ├─ blocs                    # Contains all the blocs of the project
│  │  └─ login 
│  ├─ constants                # Contains all the constants of the project
│  ├─ cubits                   # Contains all the cubits of the project
│  │  ├─ internet
│  │  └─ logout
│  ├─ firebase_options.dart    # Contains all the firebase options of the project
│  ├─ main.dart                # Entry point of the project
│  ├─ models                   # Contains all the models of the project
│  │  ├─ user_details.dart
│  │  └─ user_details.g.dart
│  ├─ pages                    # Contains all the pages of the project
│  │  ├─ basePage.dart
│  │  ├─ landingPage.dart
│  │  └─ signinPage.dart
│  ├─ repos                    # Contains all the repos of the project
│  │  └─ authRepo.dart
│  ├─ routes                   # Contains all the routes of the project
│  │  ├─ router.dart
│  ├─ services                 # Contains all the services of the project
│  │  └─ auth_service.dart
│  └─ widgets                  # Contains all the widgets of the project
│     ├─ inputField.dart
│     ├─ loginWidgets
├─ pubspec.lock
└─ pubspec.yaml
```
The application will follow the bloc design pattern.

##### Backend:
The backend will be written in Python using Django as the framework. The application architecture will be as follows:x

```
car_wash
├─ car_wash/ <-- main Django project
├─ carWash/
├─ authentication/. <-- local auth as well as socal auth
└─ manage.py
```
### RoadMap
### User Flows and Transactions
- [x] Authentication
- [ ] Admin Flow
    - [ ] Jobs
    - [ ] fulfilment
    - [ ] Post fulfilment
- [ ] User Flow
    - [ ] Product Catalog
    - [ ] Search
    - [ ] Order 
        - [ ] creation
        - [ ] Confirmation
        - [ ] fulfilment
    - [ ] Payment
    - [ ] fulfilment
    - [ ] Post fulfilment


### Process Flows
1. User Onboarding

2. Vendor KYC


### 







<!--- Eraser file: https://app.eraser.io/workspace/18LcLCCLXKxtpvQ6rrf4 --->