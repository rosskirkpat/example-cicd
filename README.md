# example-cicd


This repository contains diagrams, documents, and their associated resource links for an example of a multi-tiered environment CI/CD.


Strategy: 

Release Flow, a Trunk-based development strategy donned by Microsoft's Visual Studio Team Services (VSTS) team

10k Foot View Concepts:

- Production is isolated from new feature work and development
- Merge approved changes without needing to deploy - > Allows dev teams to maintain velocity 
- Commits during sprints become release branches cut from `main` branch once the sprint is complete, then deployment commences
- Only one release branch deployed in production at a time
