# What's New in Liferay 6.2 APIs? [](id=whats-new-in-liferay-6-2-apis-liferay-portal-6-2-dev-guide-en)

---

![Note](../../images/tip-pen-paper.png) This chapter is under construction. 

---

Liferay Portal 6.2 offers a host of new features and updates to the previous
release. Our guide to *Using Liferay Portal 6.2* shows you how to use these
features and updates and this guide shows you how to leverage them in the
applications you develop. In this chapter, we want to highlight some of the
changes to Liferay Portal's application programming interface (API). We've added
APIs for the new features and improved APIs for previously existing features. In
some cases we've modified portal's API and removed previously deprecated
interfaces. The Javadoc for Liferay Portal's entire API is availabe at
[http://docs.liferay.com/portal/6.2/](http://docs.liferay.com/portal/6.2/). But
we'll describe some of the most notable additions and changes here in this
chapter. 

<!-- NOTE TO CONTRIBUTERS

Add content describing your API changes. If a section related to your feature
already exists, integrate your content with that section. Otherwise, add a new
section and content describing the your feature's new or modified API. 

Example,

    ## FeatureXYZ

    Describe your new/modified API here

Briefly describe your feature, even if it's an existing feature from the
previous release. Explain what the new or modified API does and whow how to use
it by way of code example. 

If your feature is already described in another chapter of the Dev Guide,
consider describing the API change there instead of here in this chapter. If you
do write about it in another chapter, still mention the API change here in this
chapter and refer to that other chapter and section. That way readers can locate
your API change description from this chapter.

Example,

    ## Message Bus FeatureXYZ

    You can now to x, y, and z in the Message Bus API. See the Using Message Bus
    section of Chapter 6 for details. 

-->

## Application Display Templates [](id=application-display-templates-liferay-portal-6-2-dev-guide-14-en)

Display Settings are the simpliest way to customize the portlet display. Unlike themes or hooks, they don't require deployment and they affect specific portlet instances. But, they are limited to those that come out of the box... Wouldn’t it be great to have as many of them as we wanted? As a user, this would simplify the task of customizing the portlet display. And as developers, we wouldn’t have to change our portlet configuration code every time a new setting is required.
 
That’s exactly what Application Display Templates provides: Adding custom display settings to our portlets. Actually, this is not a new concept in Liferay. In some portlets such as Web Content, Document and Media or Dynamid Data Lists we can add as many display options (or templates) as we want.

You can use the Application Display Templates API to add this new feature to your plugins.

### Using the Application Display Templates API [](id=application-display-templates-api-liferay-portal-6-2-dev-guide-14-en)

1. Register your custom PortletDisplayTemplateHandler

   To join the exclusive ADT club your portlet has to sign a contract committing itself to fulfill all the Application Display Templates features. In other words, you have to create a your own PortletDisplayTemplateHandler implementation by extending the BasePortletDisplayTemplateHandler methods. You can check the TemplateHandler interface javadoc to learn what every method is for.
 
   Once you've created your handler, you have to declare it in the right section of your liferay-portlet.xml:

    	<?xml version="1.0"?>
    	<!DOCTYPE liferay-portlet-app PUBLIC "-//Liferay//DTD Portlet Application 6.2.0//EN" 
    	"http://www.liferay.com/dtd/liferay-portlet-app_6_2_0.dtd">
    
    	<liferay-portlet-app>
    		<portlet>
    	 		<portlet-name>MyApp</portlet-name>
    			<template-handler>
    	 			org.my.app.template.MyAppPortletDisplayTemplateHandler
    	 		</template-handler>
    		</portlet>
    	</liferay-portlet-app>

2. Declare permissions

   The action of adding Application Display Templates is new to your portlet, so you want to be sure you can grant specific permissions for it. Just add this line to your resource actions file:

    	<?xml version="1.0"?>
    	<!DOCTYPE resource-action-mapping PUBLIC "-//Liferay//DTD Resource Action Mapping 6.2.0//EN" 
    	"http://www.liferay.com/dtd/liferay-resource-action-mapping_6_2_0.dtd">
    	
    	<resource-action-mapping>
    		<portlet-resource>
    	 		<portlet-name>MyApp</portlet-name>
    	 		<permissions>
    	 			<supports>
    	 				<action-key>ADD_PORTLET_DISPLAY_TEMPLATE</action-key>
    				<supports>
    	 		<permissions>
    		<portlet-resource>
    	<resource-action-mapping>
	 
3. Add display settings configuration

   Let's move to the frontend side of your portlet. Now your portlet officially supports Application Display Templates, you'll want to expose this option to your users. Just include the liferay-ui:ddm-template-selector taglib in your portlet configuration view providing the required information, like this:
 
    	<aui:form action="<%= configurationURL %>" method="post" name="fm">
    	 	<aui:fieldset> 
    	 
    		<%
    		 TemplateHandler templateHandler = TemplateHandlerRegistryUtil.getTemplateHandler(MyType.class.getName());
    		%>
    		  
    		<liferay-ui:ddm-template-selector
    			classNameId="<%= PortalUtil.getClassNameId(templateHandler.getClassName()) %>"
    			displayStyle="<%= displayStyle %>"
    			displayStyleGroupId="<%= displayStyleGroupId %>"
    			refreshURL="<%= PortalUtil.getCurrentURL(request) %>"
    			showEmptyOption="<%= true %>"
    		/> 
    
    		</aui:fieldset>
    	 </aui:form>
 
4. Render Application Display Templates in your views

   Last but not least, you have to extend your view code to render it with the selected Application Display Template. Here is where you decide exactly which part of your view will be rendered by the Application Display Template and what will be available in the template context.
 
    	<%
    	List<MyType> myList = getMyList();
    
    	long ddmTemplateId = PortletDisplayTemplateUtil.getPortletDisplayTemplateDDMTemplateId(
    	displayStyleGroupId, displayStyle);
    	Map<String, Object> contextObjects = new HashMap<String, Object>();
    	contextObjects.put("myExtraObject", someExtraObject);
    	%>
    
    	<c:choose>
    		<c:when test="<%= portletDisplayDDMTemplateId > 0 %>">
    			 <%= PortletDisplayTemplateUtil.renderDDMTemplate(pageContext, ddmTemplateId, myList, contextObjects) %>
    		</c:when>
    		<c:otherwise>
    	 		<%= //Default view code %>
    	 	</c:otherwise>
    	</c:choose>


### Recommendations [](id=adt-recommendations-liferay-portal-6-2-dev-guide-14-en)

As we have seen, Application Display Templates bring a great power. But if there's something we've learnt, is that with great power, comes great responsability!  Let’s go through some good practices in ADT design:
 
#### Security [](id=adt-security-liferay-portal-6-2-dev-guide-14-en)
You may want to hide some classes or packages from the template context, to limit the operations that ADTs can perform on your portal. Liferay provides some portal  properties to define the restricted classes, packages and variables:

	freemarker.engine.restricted.classes= 
	freemarker.engine.restricted.packages=
	freemarker.engine.restricted.variables=serviceLocator 
	velocity.engine.restricted.classes= 
	velocity.engine.restricted.packages=
	velocity.engine.restricted.variables=serviceLocator

#### Performance [](id=adt-performance-liferay-portal-6-2-dev-guide-14-en)
Application Display Templates add extra processing task in portlet render. This inevitably has effect in the performance. To reduce this effect, make your templates as minimal as possible: focus on the presentation and use the existing API for complex operations. The best way to make efficient Application Display Templates is to know your template context well and what you can use from it. Now you don’t need to know them by heart thanks to the advanced tempalte editor! Finally, don't forget running performance tests and tuning the template cache options:

	freemarker.engine.resource.modification.check.interval=60 
	velocity.engine.resource.modification.check.interval=60

No matter which Liferay APIs you're using, you'll need to understand Liferay's
deprecation policy. That way you'll know when methods from our API's are
deprecated, and you can make any necessary changes. We'll describe the
deprecation policy next. 

## Liferay's Deprecation Policy [](id=liferays-deprecation-policy-liferay-portal-6-2-dev-guide-02-en)

Methods in Liferay's APIs are deprecated when they're no longer called by
Liferay internally. Method deprecation occurs during major and minor releases of
Liferay. A change in the first or second digits of consecutive Liferay releases
indicates a major or minor release, respectively. For example, the release of
Liferay Portal 6.2.0 after 5.2.0 was a major release; whereas the release of
6.2.0 after 6.1.30 was a minor release. Major and minor releases can have API
deprecations. 

APIs should not be deprecated between mainenance releases. Mainenance releases
are signified by a change in the third digit of the release number. For example,
the release of Liferay Portal 6.1.30 after 6.1.20 was a maintenance release and
therefore should have no API deprecations. 

To understand Liferays releases, see [Using Liferay Portal
6.2](https://www.liferay.com/documentation/liferay-portal/6.2/user-guide/-/ai/understanding-liferays-releases-liferay-portal-6-2-user-guide-15-en)

## Summary [](id=summary-liferay-portal-6-2-dev-guide-14-en)

That about wraps up our chapter on Liferay's APIs. Next, we'll reflect on what
we've learned in this guide and conclude our journey together. 
