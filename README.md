# Cytomine front-end

## Bachelor Thesis Fork
This is a fork of the Cytomine-Web-UI. 
We are working on a Bachelors thesis where we are implementing a quality assurance tool into Cytomine.
The purpose of the tool is to let patholigsts compare different images and give them a score
based on their quality for the purpose of diagnosis.


## Installation
This section assumes that you have access to an instance of Cytomine (especially [Cytomine-core](https://github.com/cytomine/Cytomine-core)) and an installed proxy. These two conditions are filled if you have used [Cytomine-bootstrap](https://github.com/cytomine/Cytomine-bootstrap). The following steps will allow you to run a front-end interacting with this instance:

* Execute `npm install` to install the dependencies ;
* Change the value of constants `CYTOMINE_CORE_HOST` and `CYTOMINE_UPLOAD_HOST` in src/utils/constants.js ; you may also update other constants according to your Cytomine instance or your preferences;
* Execute `npm run build` to build all the files into a `dist` folder.
* Configure your proxy to redirect http request on the index.html file or replace the dist folder in your Cytomine-bootstrap by this one.

## Local installation
This section assumes that you have access to an instance of Cytomine (especially [Cytomine-core](https://github.com/cytomine/Cytomine-core)). The following steps will allow you to run locally a front-end interacting with this instance:

* Execute `npm install` to install the dependencies ;
* Change the value of constants `CYTOMINE_CORE_HOST` and `CYTOMINE_UPLOAD_HOST` in src/utils/constants.js ; you may also update other constants according to your Cytomine instance or your preferences;
* Execute `npm run serve` to start a web server serving the front end (by default on `localhost:8080`).

### Remarks:

* If the front-end is not expected to run on the same host/port as the core, NGINX should be configured to allow CORS addressed to the core
* When using our software, we kindly ask you to show our website URL and our logo in all your work (web site, publications, studies, oral presentations, manuals, ...). If you use Cytomine for scientific purpose, please cite Marée et al., Bioinformatics 2016 as reference paper. See our license files for additional details.
  - URL: http://www.cytomine.org/
  - Logo: [Available here](https://doc.cytomine.org/img/logo_cyto_org.png)
  - Scientific paper: Raphaël Marée, Loïc Rollus, Benjamin Stévens, Renaud Hoyoux, Gilles Louppe, Rémy Vandaele, Jean-Michel Begon, Philipp Kainz, Pierre Geurts and Louis Wehenkel. Collaborative analysis of multi-gigapixel imaging data using Cytomine, Bioinformatics, DOI: 10.1093/bioinformatics/btw013, 2016. http://dx.doi.org/10.1093/bioinformatics/btw013

## Code of Conduct

We subscribe to the [Contributor Convenant Code of Conduct](https://documentation.cytomine.org/Code-of-Conduct) for a respectful community.
