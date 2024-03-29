# Weekly Report Automation


This repository includes material to automate weekly reporting from all countries to the bureau. It compiles information from each operation in the region based on predefined structure within a single report.


## Form

An [xlsform questionnaire](http://xlsform.org) is used in [Kobotoolbox server](http://kobo.unhcr.org) - [`xlsform.xlsx`](https://github.com/unhcr-americas/weekly-report/raw/main/xlsform.xlsx). You can see a preview/demo of the form - __this is NOT the production /official FORM__ here: https://enketo.unhcr.org/single/6o7btvE1 


## Report Template

Based on this form, a report template was created. The Rmd files `report.Rmd` pull data from the Kobo API in order to automate the production of weekly report.

to run the report, you will need to install 

```{r}
## Basic todyverse packages
install.packages("tidyverse")
install.packages("lubridate")
install.packages("httr")

## Package for UNHCR report template
# install.packages("devtools")
devtools::install_github("vidonne/unhcrdown")

## Package to create automatation with http://rstudio.unhcr.org
devtools::install_github("rstudio/rsconnect")

```

You will also need to add to your `.Renviron`, your kobotoolbox token `KOBO_API_KEY=xxxmytokenxxxx`. The .Renviron file is a way to store sensitive information such as passwords or API keys. All the information in these files are stored as environment variables and are enabled when you start a session. It’s a common thing in computer science: a value pair usually set outside the program, often built into the operating system. You edit direcly this file with 

```{r}
usethis::edit_r_environ()

```


Basically the report works as follows: 
 * in a first chunk the report pull the data from kobo API, using the token that was just set up and 
 * some basic reshaping are performed.  
 * then 3 functions are created to parse the data and according to sub-region(`render_region`)), country(`render_country`) and section (`render_section`). 
 * Data are then filtered for the specific reporting week and then a last function (`purr::walk(levels(fct_drop(datanow$region)), ~render_region(datanow, .))` - see [doc](https://purrr.tidyverse.org/reference/map.html) ) is used to loop around the content. 


The front cover image cover (default is `cover_grey.jpg`) can be changed as required. The best source of image is [UNHCR media library](http://media.unhcr.org).

```
    front_img: cover_grey.jpg
    
```

## Automation of Report Generation

The Rmd is knotted on a weekly basis and the files shared via various channels - email/teams/sharepoint

It is set up here to allow for automation with a direct connection to UNHCR kobo server using an authentication token.

Below is a step by step how to in order to configure the automation.

 
### Step 1 - Set up and/or refresh the `manifest.json`

You first need to create a documentation file - which allow the Rstudio server to regenerate your report. This files is created by th e `rsconnect::writeManifest`.

Note that if you are the one developing any of this package, you will first need to re-install them from github or gitlab for the manisfest file to be correctly written. See more documentation here: [Git Backed Content - RStudio Connect: User Guide](https://docs.rstudio.com/connect/user/git-backed/)

```{r}
rsconnect::writeManifest(appPrimaryDoc = "report.Rmd")
```

### Step 2 -  Publish the report to Rstudio Connect from Github

Go to UNHCR Rstudio server - [http://rstudio.unhcr.org](http://rstudio.unhcr.org) - you need first to have a license associated to your account - Contact Global Data Service Data Science team for that.
 
![ ](https://raw.githubusercontent.com/unhcr-americas/weekly-report/main/inst/fromGit.png) 



![ ](https://raw.githubusercontent.com/unhcr-americas/weekly-report/main/inst/fromGit2.png)


![ ](https://raw.githubusercontent.com/unhcr-americas/weekly-report/main/inst/fromGit3.png)



### Step 3 -  Set up your kobotoolbox API key within Rstudio Connect

You need now to set up the kobotoolbox authentication token within the Rstudio server so that the server can actually pull the data from Kobotoolbox in order to regenerate the Report.

![ ](https://raw.githubusercontent.com/unhcr-americas/weekly-report/main/inst/fromGit4.png)

![ ](https://raw.githubusercontent.com/unhcr-americas/weekly-report/main/inst/fromGit5.png)


![ ](https://raw.githubusercontent.com/unhcr-americas/weekly-report/main/inst/fromGit6.png)


### Step 4 -  Set up report frequency generation and sending it to your email


![ ](https://raw.githubusercontent.com/unhcr-americas/weekly-report/main/inst/fromGit7.png)

et voila...



### Step 5 -  Use power automate to forward automatically the email with the report to the data supervisor
 
[Microsoft Power Automate](https://make.powerautomate.com/) is a convenient way to automatically re-forward the generated report to the trget audience.



----

## Todo: automate email notification with  Blastula

see documentation here: https://posit.co/blog/emails-from-r-blastula-0-3/ 

In order to implement this, a request to UNHCR ICT admin is required so that email can be send over API.

Further exploration is required to ensure token authenciation - see https://cran.r-project.org/web/packages/Microsoft365R/vignettes/auth.html 


```{r} 
install.packages("blastula")
install.packages("Microsoft365R")
## see some doc here - blastula::prepare_rsc_example_files()

library(Microsoft365R)

# The AzureR packages can save your authentication credentials in a directory
# This saves you having to re-authenticate with Azure in future sessions. 
# Create this directory? (Yes/no/cancel) 

## note that you will get here a token to be re-used afterward..

outlb <- get_business_outlook()

# compose an email with blastula
library(blastula)
bl_body <- "## Hello!
Please find attached the weekly report for the Region."

bl_em <- compose_email(
    body=md(bl_body),
    footer=md("UNHCR Reporting Unit")
)
em <- outl$create_email(bl_em, 
                        subject="Weekly Report",
                        to="diffusion-list@unhcr.org")

# add an attachment and send it
em$add_attachment("report.html")
em$send()

```



