---
layout: post
title:  "Association with waterline for storing large ammount of text"
date:   2015-06-10 00:51:02
categories: waterline sailsjs mysql performance
---

## **Introduction:**    


One-To-One associations are associations between exactly two models connected by a single and unique foreign key.

###**Example:**  

here's an example to define one to one relationship for a large ammount of data article

![example]({{ site.url }}/images/article.png)

Article table is used to store the frequent use data like article and vote number, while Article_content store the Article detail . Both tables contains the same article_id as primary key. And, in Article_content table, Article_id is the primary key and also a foreign key to Article table.

now implementing our models:  
Article_content.js  <br/> 
{%highlight javascript%}
/**
* Article_content.js
*/

module.exports = {

    
  identity: 'Article_content',
  //false to prevent creating createdAt and updatedAt
  autoCreatedAt:false,
  autoUpdatedAt:false,
  tableName:'Article_Content',
  autoPK:false, //false to prevent creating id

  attributes: {
  	Content:{
  		type:'text'
  	},
  	shown_content:{
  		type:'text'
  	},
  	id_article:{
  		primaryKey:true,
  		foreignKey:true,
  		references:'Article',
  		on:'id',
  		required:true
  	}

  }
};
{%endhighlight%}  <br/>    

Article.js    
 <br/>
{%highlight javascript%}
/**
* Article.js
*/

module.exports = {


  
  identity: 'Article',
  autoCreatedAt:false,
  autoUpdatedAt:false,
  tableName:'Article',

  attributes: {
  	Vote:{
  		type:'integer'
  	},
  	date_pub:{
  		type:'datetime'
  	},
  	writer:{
  		//model:'Utilisateur',required: true /*relation with the author*/
  	}
  	

  }  
{%endhighlight%}  
 <br/>

##__why i split the article into two tables?__  

as you can see i used type:'text' for the Content attribute for more details ,please
read [this article][article] .  
for matter of performance it is recommended to decouple the tables , actualy  the storage engine reserve the maximum space for field for example InnoDB stores fixed-length, such as Char(N) or text.  
 to store UTF-8 text : such column occupy 3*L+2 byte  
*L is the length of a given string L<2^16   



 <br/> <br/>

 [article]:http://dev.mysql.com/doc/refman/5.5/en/blob.html