* IDE : Eclipse

Cannot change version of project facet Dynamic Web Module to 2.5.
Dynamic Wem Module을 2.5 버전으로 바꾸려고 하지만 할 수 없다는 에러다.

Soluton 1
1. 해당 프로젝트 workspace/../org.eclipse.wst.common.project.facet.core.xml을 열어본다
2. jst.web 버전이 2.5로 설정되었는지 확인한다.
3. 아니라면 원하는 Dynamic Web Module 버전으로 설정하고 저장, 프로젝트 재시작

Solution 2
1. web.xml에 정의된 XML 스키마 버전을 확인한다.
2. 해당 프로젝트에서 사용하는 Tomcat 버전에 맞는 서블릿 스펙으로 변경

http://tomcat.apache.org/whichversion.html
해당 링크에서 Tomcat 버전에 맞는 서블릿 스펙을 확인가능
