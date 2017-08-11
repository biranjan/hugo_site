---
title: How to take your predictive model to next Level using Shiny
author: ~
date: '2017-08-06'
slug: how-to-take-your-predictive-model-to-next-level-using-shiny
categories: ["shiny","R"]
tags: ["glm","highchater","shiny","R"]
draft: no
---

Building a predictive model must be one of the usual task for students to do in their machine learning or predictive modelling class.However, when I was student I often had lingering question what to do with model now ? In this post I will talk about how to take your predictive model to the next level and make a web app where other people
can interact with your model. The final outcome will look like [this] (https://biranjan.shinyapps.io/titanic_shinnyapp/). If the apps is not available due to the free account limit clone it from my git hub repository from [here] (https://github.com/biranjan/titanic_shinnyApp) and run it locally. 

![screen shot of app](http://res.cloudinary.com/dstcxl6nt/image/upload/c_scale,h_400,w_500/v1502471534/titanic_shinyApp_kujeym.png "screen shot")

## Designing  Predictive Model
The data used here is from the Titanic data set which can be downloaded from [kaggle] (https://www.kaggle.com/c/titanic). Probably there you can learn few tricks on making your model accuracy higher I think highest accuracy might be almost 99 %. Here, I am using simple *glm* model to predict the survival rate using five explanatory variable passenger class(Pclass),Sex,Age, port of embarkation(Embarked), title of person (Title). I think the accuracy is about 70%. 


```r
# if you wnat to know more about glm 
?glm()

# loading the data
mydata <- read.csv("Data/Titanic_data.csv")

# making predictive model
# try to validate by partitioning your dataset into test and train 
log_mod <- glm(Survived~Pclass+Sex+Age+Embarked+Title,
               data=mydata,family = "binomial")

```

Predicting from model is really simple once you have the model you feed a new data in the model. You need to make sure that data types are the same as previously used in the model otherwise it might produce an error.

```r
# Creating new data to predict on
# See the data description to know about the variable
Pclass = c(1)
Sex = c("Male")
Age = c(25)
Embarked = c("C")
Title =  c("Mr")

# loading the data
mydata <- read.csv("Data/Titanic_data.csv")

# making your predictive 
# try to validate by partitioning your dataset into test and train 
log_mod <- glm(Survived~Pclass+Sex+Age+Embarked+Title,
               data=mydata,family = "binomial")

```

## The next Step 
The next step includes wrapping the model in web application. Here we use shiny web framework for R. It is as easy going to rstudio and clicking on shiny web app from file menu. Here, I assume that you know something about shiny. Shiny basically needs Ui file and server file, In Ui we define how things are shown and in server we define the logic and operations. 

### Working in Ui
In file Ui.R we will define input elements ( see [here examples] (http://shiny.rstudio.com/gallery/widget-gallery.html) if you are not familiar with them) where user can input the Pclass,gender,age,title and embarkation place. And we use that to predict whether they are going to die or survive. 
Shiny has predefined input elements so we use that and we put them in sidebar. Once we have the input element ready we can move to Server.

```r

 sidebarLayout(
        sidebarPanel(
            helpText(("Your information")),
            
            radioButtons("Title",
                         label = "title",
                        choices = list("Mr","Miss","Other"),
                        selected = "Mr"),
            
            textInput("Name",
                      label = "Your name"),
            
            selectInput("Sex",
                        label = "Select your gender",
                        choices = list("male","female"),
                        selected = "male"),
            
            radioButtons("Pclass",
                        label = "Select your Passenger class",
                        choices = list("choice 1"=1,"choice 2"=2,
                                       "choice 3"=3,"choice 4"=4)),
            
            numericInput("Age", 
                         label = ("Select your Age"),  
                         value = 20),
            
            
            radioButtons("Embark",
                         label = "Select the port of embarkation",
                         choices = list("Cherbourg"="C","Queenstown"="Q","Southampton"="S"),
                         selected = "C" ),
            
            actionButton("Confirm",
                            "Confrim ticket")
                         
            
        )

```

### Working in Server
You can save your model as R object and load in R that way you don't need the data file and run the glm function but here I am going to load the data and create glm model in server.If you data is big than we might want to rethink this approach. In server we pull the variable that have been inserted through ui and use them to predict the survival rate. 

Here we are using *highcharter* package to build interactive pie chart displaying your chances of survival. And I also added if statement to display text saying whether you might survive or not based on the probability gained from predict function, it simply checks whether the probability is higher that 50 % or not.

```r
shinyServer(function(input, output) {
    
    
  observeEvent(input$Confirm, {
   
   # getting the varibale from ui input 
   
   Pclass <- as.numeric(input$Pclass)
   Age <- input$Age
   Sex <- input$Sex
   Title <- input$Title
   Embarked <- input$Embark
  
   # creating a data frame from the input
   newdat1 <- data.frame(Pclass,Sex,Age,Embarked,Title)
   # predicting on new data
   pred1 <- predict(log_mod,newdata = newdat1,type = "response")
   # making a data frame to make a pie chart
   pie_frame <- data.frame(pie = c(round(pred1,2),round(1-pred1,2)))
   
   # rendering pie chart
   output$dem_pie <- renderHighchart({
     
       hchart(type="pie",pie_frame, hcaes(y=pie,x=c("Alive","Dead")),name="Total",colors=list("lightgreen","tomato")) 
       
   })
   
   output$pred <-  renderText({
        if(pred1 > 0.5) {
        paste("Helo ",input$Name, "! Yout might just survive the journey")
        } else {
          paste("Helo ",input$Name,"You might not survive the jounrney")
        }
          })
      
    })

```

### Back Again to Ui 
Since we crated some elements to display on ui from server so we need to place those objects in Ui. Here we are going to put the elements fro server in main panel. 
```r
 mainPanel(
            
            div(id = "panel",textOutput("text2"),
            div(uiOutput("pred"))),
            div(id = "placeholder"),
            highchartOutput("dem_pie"),
            img(img(src="img/pic.png",width="600px"))
            )

```

## We Are Done 
To see the full code please head over the to my git hub [repository] (https://git hub.com/biranjan/titanic_shinnyApp). This one simple approach to use your model in a meaningful way and it is fun for yourself and hopefully others too to interact with the model you built. If the model is not complex wrapping it in shiny shouldn't be hard.