# Einführung in CDI

Unter dem Namen "Contexts and Dependency Injection for the Java EE platform", kurz CDI, wurde am 10. Dezember 2009 eine neue Spezifikation in der finalen Version veröffentlicht, welche schon bald darauf das Java EE Ökosystem nachhaltig verändern sollte. Inspiriert von erfolgreichen Open-Source Frameworks \(wie bspw. Spring, Seam, Guice\) und der damit verbundenen langjährigen Erfahrung wurde ein neues typsicheres Komponentenmodell spezifiziert, welches die tägliche Arbeit erheblich erleichtert. Doch auch komplexere Anforderungen kommen dank der Flexibilität von CDI nicht zu kurz. Diese Flexibilität ermöglichte portable CDI Erweiterungen, welche entscheidend zum Erfolg von CDI beigetragen haben.

**Ziel dieses Buchs    
**

In diesem Buch lernen Sie Schritt für Schritt die Grundkonzepte von CDI und wie Sie sowohl Java SE als auch Java EE Projekte erfolgreich mit diesem neuen Komponentenmodell umsetzen können. Wie bei jeder Technologie gibt es auch bei CDI den einen oder anderen Fallstrick. Durch Tipps und Tricks lernen Sie diese zu erkennen und erfahren Details zu den Lösungsmöglichkeiten. Neben der Integration mit anderen Technologien widmen wir uns auch erfolgreichen CDI-Erweiterungen und zeigen Ihnen auf wie Sie von diesen profitieren können.

## Context- und Dependency-Management

Context-Management: Um zu verstehen warum mit der Arbeit an CDI begonnen wurde, machen wir einen kurzen Ausflug in die Anfänge der Softwareentwicklung mit Java.

Ohne ein zusätzliches Framework steht Ihnen der rudimentäre Sprachumfang von Java zur Verfügung. Geht es um die Erzeugung neuer Instanzen einer Klasse, so können Sie das Schlüsselwort new verwenden. Allerdings müssen Sie eine solche Instanz auch verwalten, um sie zu einem späteren Zeitpunkt wieder verwenden zu können \(= Context-Management\). Ähnlich wie in anderen Programmiersprachen haben sich auch in Java hierfür verschiedene Entwurfsmuster etabliert. Doch selbst diese Entwurfsmuster führten in großen Applikationen oft zu unnötig komplexen oder ausschweifenden Umsetzungen. Um Instanzen effizienter zu verwalten und mit zusätzlicher Funktionalität anzureichern wurden bald Komponentenmodelle eingeführt, mit welchen die Komplexität in Konfigurationsdateien ausgelagert wurde. Mangels einer sinnvolleren Alternative wurde hierfür oftmals XML als Konfigurationsformat herangezogen. Dies führte jedoch zu neuen Herausforderungen. Ein zentraler Teil einer Applikation ist nicht mehr typsicher und die Konfiguration mancher Komponentenmodelle wurde so umfangreich, dass diese oft mit Generatoren erzeugt werden musste. Im Laufe der Zeit wurden die Komponentenmodelle besser und der Konfigurationsaufwand erheblich gesenkt. Jedoch basiert das Format der Konfigurationsdateien weiterhin auf Strings, wodurch viele Vorteile einer typsicheren Sprache wie Java verloren gehen.

Eine zweite Herausforderung bei der Entwicklung von Applikationen sind die Referenzen zu anderen Instanzen \(= Dependency-Management\). Hierbei erhöht sich die Komplexität, sobald Beans mit unterschiedlicher Lebensdauer \(Scopes\) beteiligt sind. Sorgt der Container dafür, dass die erforderlichen Referenzen zu anderen Instanzen automatisch gesetzt \(= injiziert\) werden, so spricht man von Dependency-Injection. Für dieses Dependency-Management nutzen viele Komponentenmodelle Konfigurationseinträge, welche ebenfalls zum Verlust der Typsicherheit führen.

Ein Container stellt ein Komponentenmodell zur Verfügung, welches vom Applikationscode verwendet werden kann, um Teile der Applikation einfacher umzusetzen. Dadurch stellen sie das Bindeglied zwischen der darunter liegenden Laufzeitumgebung und dem Applikationscode dar.Sowohl beim Context- als auch beim Dependency-Management gab es bis zu JDK 5 keine Alternative, um diese Grundproblematik effizienter und vor allem typsicher zu lösen. 2004 wurden die anfänglich stark unterschätzten Annotationen mit Java 5 eingeführt. Doch erst 2006 wurden mit Java EE 5 Annotationen erstmals \(offiziell\) für Dependency-Injection verwendet. Im darauffolgenden Jahr wurde dieser Ansatz von einem Projekt namens Guice bezüglich der Typsicherheit verfeinert. Durch die zunehmende Beliebtheit von typsicheren Komponentenmodellen war eine Spezifikation solcher Konzepte naheliegend. Ursprünglich wurde CDI \(JSR-299\) unter dem Namen "Web Beans" geführt. Es war nämlich das primäre Ziel ein Bindeglied zwischen der JSF- und EJB-Spezifikation zu schaffen. Drei Jahre später feierte CDI sein Debüt. Bis zur finalen Version der Spezifikation hatte sich nicht nur der Name der Spezifikation geändert, sondern es wurden mehrere größere Überarbeitungen vorgenommen. So wurde bspw. ein Teil der Spezifikation ausgelagert \(zu JSR-330\) und stellt nicht nur die Basis für CDI dar, sondern auch für andere Komponentenmodelle. Außerdem ist CDI in der heutigen Form nicht mehr an Java EE gebunden, sondern kann ebenfalls problemlos in Java SE Applikationen verwendet werden.

> Tipp: JSR-330 besteht aus 5 Annotationen und einem Interface und spezifiziert einen minimalen Funktionsumfang, welcher für Dependency-Injection und die Definition eigener Scopes erforderlich ist. Neben Implementierungen der CDI-Spezifikation wird die Spezifikation auch von anderen Projekten als Basis verwendet.

## Annotationen als zentraler Bestandteil

Annotationen sind zusätzliche Metadaten und wurden in Java 5 als Erweiterung des Typsystems eingeführt. Da CDI hauptsächlich auf Annotationen basiert und die Erstellung eigener Annotationen zum täglichen Handwerkszeug für die Arbeit mit CDI gehören, sehen wir uns ein paar Details etwas näher an. Bei Annotationen muss grundsätzlich unterschieden werden, ob sie zur Laufzeit der Applikation abgefragt werden können oder nicht. Ein bekannter Vertreter für Annotationen, welche nicht zur Laufzeit abgefragt werden können ist @Override , da hier @Retention\(RetentionPolicy.SOURCE\) definiert ist. Für ein Komponentenmodell wie CDI sind natürlich nur Informationen sinnvoll, welche zur Laufzeit zur Verfügung stehen. Daher müssen alle Annotationen, welche Sie in Zusammenhang mit CDI erstellen, immer @Retention\(RetentionPolicy.RUNTIME\) definieren. Anderenfalls sieht der CDI-Container die Annotation zur Laufzeit nicht.

Darüber hinaus muss noch angegeben werden an welchen Stellen die Annotation verwendet werden kann. Dies wird mit Hilfe von **@Target** ausgedrückt. Somit können wir bereits die gebräuchlichste Annotation namens @Inject für die tägliche Arbeit mit CDI, welche in Listing [Spezifizierter Aufbau von @Inject](#spezifizierter-aufbau-von-inject) dargestellt ist, analysieren.

###### Spezifizierter Aufbau von @Inject

```
@Target({ElementType.CONSTRUCTOR,
    ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Inject {}
```

Im Falle von @Inject sind die möglichen Verwendungsziele Konstruktoren, Methoden und Felder. Abgesehen von den eben erwähnten Elementtypen sind im Kontext von CDI noch ElementType.TYPE für Annotationen auf Klassenebene, ElementType.PARAMETER für Annotationen auf Methodenparametern und ElementType.ANNOTATION\_TYPE für Annotationen auf Annotationen wichtig. Eine Annotation zu annotieren mag sich anfänglich etwas ungewohnt anhören. Im Laufe des Buches werden wir jedoch sehr sinnvolle Verwendungsmöglichkeiten für diesen Elementtyp kennenlernen.

## Hello CDI



