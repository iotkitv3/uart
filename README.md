## UART (Universal Asynchronous Receiver Transmitter)
***

> [⇧ **Home**](https://github.com/iotkitv4/intro)

![](https://raw.githubusercontent.com/iotkitv4/intro/main/images/UART.png)

Das untere Diagramm zeigt die dazu invertierten Spannungspegel auf der RS-232-Schnittstelle. [Quelle Wikipedia](http://de.wikipedia.org/wiki/Universal_Asynchronous_Receiver_Transmitter)

- - - 

Universal Asynchronous Receiver Transmitter, kurz UART, ist eine elektronische Schaltung, die zur Realisierung digitaler serieller Schnittstellen dient. Dabei kann es sich sowohl um ein eigenständiges elektronisches Bauelement (ein UART-Chip bzw. -Baustein) oder um einen Funktionsblock eines höherintegrierten Bauteils (z. B. eines Mikrocontrollers) handeln.

Eine UART-Schnittstelle dient zum Senden und Empfangen von Daten über eine Datenleitung und bildet den Standard der seriellen Schnittstellen an PCs und Mikrocontrollern. Auch im industriellen Bereich ist die Schnittstelle mit verschiedenen Interfaces (z. B. [RS-232](http://de.wikipedia.org/wiki/RS-232) oder [EIA-485](http://de.wikipedia.org/wiki/EIA-485)) sehr verbreitet.

Die Daten werden als serieller digitaler Datenstrom mit einem fixen Rahmen übertragen, der aus einem Start-Bit, fünf bis maximal neun Datenbits, einem optionalen Parity-Bit zur Erkennung von Übertragungsfehlern und einem Stopp-Bit besteht. Um dem Empfänger eine Synchronisationszeit auf den Takt der empfangenen Daten einzuräumen, kann das Stopp-Bit auf das 1,5 oder 2-fache der normalen Übertragungszeit eines Bits verlängert werden. Das wird als 1,5 bzw. 2 Stopp-Bits bezeichnet.

**In einfachen Mikrocontroller-Systemen werden Daten häufig über UART-Schnittstellen ausgetauscht, die ohne Handshake, nur über Rx und Tx, verwirklicht sind. Diese für kurze Entfernungen geeignete, auch [CMOS-UART bzw. TTL-UART](http://de.wikipedia.org/wiki/Logikpegel) genannte Implementierung wird von praktisch allen Mikrocontrollern unterstützt und kann bei entsprechend geringen Übertragungsraten auch über Software realisiert werden.**

Um den [CMOS-UART bzw. TTL-UART](http://de.wikipedia.org/wiki/Logikpegel) Pegel für den USB Bus verfügbar zu machen, kann ein [USB Serial Adapter](http://arduino.cc/en/Main/USBSerial) verwendet werden. Boards haben diesen, in der Regel, integriert (mbed - Ja, Arduino UNO - Ja, [Arduino Mini - Nein](http://www.arduino.cc/en/Main/ArduinoBoardMini) )

*Die Standardeinstellungen für den Seriellen Port (USB) sind 9600 baud, 8 bits, 1 stop bit, no parity (aka 9600-8-N-1).*

Um Daten auf die Serielle Schnittstelle auszugeben, sind keine zusätzlichen Objekte erforderlich. Die Serielle Schnittstelle wird automatisch geöffnet und Daten können mittels [printf](http://www.cplusplus.com/reference/cstdio/printf/) ausgegeben und mittels [scanf](http://www.cplusplus.com/reference/cstdio/scanf/) ausgelesen werden.

### Anwendungen 

* Ausgabe von Debug Informationen
* Fernsteuerung des Gerätes z.B. wie bei 3D Druckern
* Board - PC Kommunikation
* Ansprechen von Modems wie [Bluetooth](http://developer.mbed.org/platforms/FRDM-K64F/#features), [WLAN](https://os.mbed.com/questions/4993/How-to-interface-esp8266-wifi-module/), [GPS](http://developer.mbed.org/cookbook/GlobalSat-EM-406-GPS-Module)

### Beispiele

Für die Beispiele muss [mbed Studio](https://os.mbed.com/studio/) oder ein Terminalemulations Programm (siehe [Links](#links)) und für Windows 10, zusätzlich ein Treiber installiert werden. 

Die Konfiguration und der Firmware Update des ESP8266 WLAN Modems ist weiter [unten](#konfiguration-esp8266) beschrieben. 

#### [Serielle Ausgabe](main.cpp) 

Demonstriert die verschiedenen Ausgabemöglichkeiten mit `printf`.

#### Serielle Eingabe

Demonstriert die verschiedenen Eingabemöglichkeiten mit `scanf`.

<details><summary>main.cpp</summary>
  
    #include "mbed.h"
    
    int i;
    float f;
    char s[6];      // max. 5 Zeichen + \0
    
    int main()
    {
        printf  (" Eingabe int: " );
        scanf   ( "%d", &i );
        // Integer: Standard, genau 4-stellig, 4-stellig mit Vornullen
        printf  ( "\nint %d, %4d, %04d\n", i, i, i );
    
        printf  ( "Eingabe float: " );
        scanf   ( "%f", &f );
        // float: Standard, Vor-/Nachkommastellen gerundet
        printf  ( "\nfloat %f, %4.2f\n", f, f );
    
        printf  ( "Eingabe string (max 5 Zeichen): " );
        scanf   ( "%5s", s );
        // String: Standard, mit Anzahl der auszugebenden Zeichen
        printf  ( "\nstring %s, %.*s\n", s, 3, s );
    
        //////////////////// String nach Integer, Float
        char sint[] = "123";
        char sfloat[] = "1.235";
    
        sscanf  ( sint, "%d", &i );
        printf  ( "int %d\n", i );
    
        sscanf  ( sfloat, "%f", &f );
        printf  ( "float %f\n", f );
    
        /////////////////// Parsen z.B. von HTTP Adressen mit Parametern
        // http://<IP-Adresse>/?servo1=0.5&servo2=0.1 scannen. sscanf liefert die Anzahl erkannter %f Argumente
        char addr[] = "GET /?servo1=0.1&servo2=0.9";
        float s1, s2;
    
        // sscanf liefert die Anzahl erkannten Argumente
        if  ( sscanf( addr, "GET /?servo1=%f&servo2=%f", &s1, &s2 ) == 2 )
            printf  ( "Servo1 %f, Servo2 %f\n", s1, s2 );
    }

</p></details> 
 
#### WLAN Modem ESP8266 lowlevel

Ansprechen des ESP8266 WLAN Modems via AT-Befehlen (nur IoTKit K64F Board). Nicht getestet!

<details><summary>main.cpp</summary>

    /** ESP 8266 WLAN Modem initialisieren 
     *  - muss als erstes erfolgen, damit die Verbindung zum AP steht
     *  - wenn das Modem nicht sauber oder zu langsam funktoniert, wait Zeiten erhoehen
    */
    #include "mbed.h"
    
    UnbufferedSerial  pc( USBTX, USBRX );
    UnbufferedSerial  dev( PTC15, PTC14 );
    // Reset Modem
    DigitalOut rst( PTA19 );
    DigitalOut led1( D11 );
    DigitalOut led4( D12 );
    
    /** Lesen von Modem, Ausgabe auf UART USB */
    void recvChar()
    {
        char c;
    
        led1 = !led1;
        // Read the data to clear the receive interrupt.
        if ( pc.read(&c, 1) ) 
            dev.write( &c, 1 );
    }
    
    /** Schreiben auf Modem ab UART USB */
    void sendChar()
    {
        char c;
    
        led4 = !led4;
        if ( dev.read( &c, 1) ) 
            pc.write( &c, 1 );    
    }
    
    /** Senden eines Strings an das Modem */
    void send( char* out )
    {
        dev.write( out, strlen( out ) );
        dev.write( "\r\n", 2 );
    }
    
    int main()
    {
        rst = 0;
        // 1. Teil: Initialisierung - Baudraten muessen identisch sein
        pc.baud( 115200 );
        dev.baud( 115200 );
        thread_sleep_for( 1000 );
        rst = 1;
    
        // Register a callback to process a Rx (receive) interrupt.
        pc.attach( &sendChar, SerialBase::RxIrq );
        dev.attach( &recvChar, SerialBase::RxIrq );
        
        send( (char *) "AT+RST" );       // RESET Modem
        thread_sleep_for( 2000 );
        
        send( (char *) "AT+GMR" );       // Ausgabe Firmware Version - optional
        thread_sleep_for( 2000 );    
        
        send( (char *) "AT+CWMODE=1" );  // Station Mode, d.h. Modem = Client zu AP
        thread_sleep_for( 1000 ); 
        
        //send( "AT+CWLAP" );     // List AP - optional
        //wait( 10.0f );     
        
        send( (char *) "AT+CWJAP=\"LERNKUBE\",\"l3rnk4b3\"" );  // Verbindung AP (SSID, PW)
        thread_sleep_for( 10000 );    
        
        send( (char *)"AT+CIFSR" );  // Ausgabe IP-Adresse
        thread_sleep_for( 1000 );
        
        // 2. Teil: Webseite von google.com holen
        send( (char *) "AT+CIPMUX=1" );  // Mehrere Verbindungen aktivieren
        thread_sleep_for( 1000 );    
        
        send( (char *) "AT+CIPSTART=4,\"TCP\",\"httpbin.org\",80" );  // Socket zu google.com oeffnen
        thread_sleep_for( 1000 ); 
        
        send( (char *) "AT+CIPSEND=4,18" );  // 18 Zeichen senden
        thread_sleep_for( 1000 );     
        
        send( (char *) "GET / HTTP/1.0\r\n" );  // HTTP GET
        thread_sleep_for( 1000 ); 
    
        exit( 0 );    
    }  
  
</p></details> 


### Links

* Terminalemulations Programme: [Windows](http://sourceforge.jp/projects/ttssh2/releases/), [Mac](http://freeware.the-meiers.org/), [Linux](http://manpages.ubuntu.com/manpages/vivid/man8/picocom.8.html)
* [PC Driver (nur Windows)](https://os.mbed.com/handbook/Windows-serial-configuration)
* [printf Reference](http://www.cplusplus.com/reference/cstdio/printf/)
* [scanf Reference](http://www.cplusplus.com/reference/cstdio/scanf/)

## Konfiguration ESP8266
***

Der IoTKit K64F wird mittels dem [ESP8266 WLAN Modem](https://de.wikipedia.org/wiki/ESP8266) ans Internet angeschlossen.

Ab mbed OS V5.10 ist der ESP8266 Driver Bestandteil von mbed und keine separate Library mehr. 

Um auf die neuste Version von mbed OS zu updaten ist wie folgt Vorzugehen:
* mbed-os - Library im Projekt löschen
* Mittels rechter Maustaste `Import Library ...` -> `From URL ...` anwählen
* Als URL [https://github.com/ARMmbed/mbed-os.git](https://github.com/ARMmbed/mbed-os.git) eintragen und Library importieren.

Der IoTKit V3 kommuniziert mittels UART mit dem [ESP8266](https://de.wikipedia.org/wiki/ESP8266) dabei werden folgende Pins verwendet:
* TX - PTC15
* RX = PTC14

Die eigentliche Konfiguration der Schnittstelle erfolgt in der Datei `mbed_app.json`. Die sehen Einträge wie folgt aus:

    {
        "config": {
            "wifi-shield": {
                "help": "Options are internal, WIFI_ESP8266, WIFI_ISM43362, WIFI_IDW0XX1",
                "value": "WIFI_ESP8266"
            },
            "wifi-ssid": {
                "help": "WiFi SSID",
                "value": "\"LERNKUBE\""
            },
            "wifi-password": {
                "help": "WiFi Password",
                "value": "\"l3rnk4b3\""
            },
            "wifi-tx": {
                "help": "TX pin for serial connection to external device",
                "value": "PTC15"
            },
            "wifi-rx": {
                "help": "RX pin for serial connection to external device",
                "value": "PTC14"
            },
            "wifi-debug": {
                "value": true
            }
        }
    }
   
Weitere nützliche Informationen zum ESP8266 sind in diesem [Blog](https://orxor.wordpress.com/2015/01/30/esp8266-intro/) zu finden. Ausserdem existiert eine [ESP8266 Gruppe](https://os.mbed.com/teams/ESP8266/).

### WLAN Modem Konfiguration (UART)

Zum Konfigurieren oder Update der Firmware eignet sich am Besten der PL2303HX Converter USB To RS232 TTL der wie folgt mit dem Modem zu verbinden ist:

*   Grünes Kabel - mit TXD0 (Transmit) verbinden
*   Weisses Kabel - mit RXD0 (Receive) verbinden

Das Rote und Schwarze Kabel wird nicht benötigt. Die Baudrate ist auf 115200 einzustellen.

**SDK Version ab 1.5.4:**

Modem mit Access Point verbinden:

*   **AT+RST** - Reboot Modem
*   **AT+GMR** - Ausgabe SW Version
*   **AT+CWMODE=1** - WLAN Modem als Client (Station) konfigurieren
*   **AT+CWJAP_CUR="mcbmobile_2EX","android%123"** - mit Access Point mcbmobile_2EX verbinden
*   **AT+CIFSR** - Ausgabe der IP-Adresse

#### Firmware Update

Dazu ist das Modem mit einem USB To RS232 TTL Converter zu Verbinden und eine Drahtbrücke von GND nach GPIO0 zu legen.

Die Software und Firmware wie in [ESP8266 Firmware Update](https://os.mbed.com/teams/ESP8266/wiki/Firmware-Update) beschrieben herunterladen.

Mittels eines Drahtes GND und RESET für eine Sekunde kurzschliessen und mittels des Flash Download Tools die Firmware updaten.

Die beschriebene Schaltung "serial passthrough" hat nicht funktioniert, so dass der Update mittels USB To RS232 TTL Converter erfolgt ist.
Auf der mbed MCU ist vorher ein einfaches Programm, z.B. DigitalOut zu uploaden, welche das ESP8266 nicht verwendet und auch nicht stört.

### Links

* [Mikrocontroller.net](https://www.mikrocontroller.net/articles/ESP8266)
* [espressif Firmware](https://github.com/espressif/ESP8266_NONOS_SDK/tree/master/bin/at)
* [Verzeichnis AT Commandos.](https://room-15.github.io/blog/2015/03/26/esp8266-at-command-reference/)
* [Chip Hersteller](https://espressif.com/en)
* [Auführliche Beschreibung](https://playground.boxtec.ch/doku.php/wireless/esp8266)

## Übungen
***

> [⇧ **Nach oben**](#)

### AT Commands

Ändern Sie das [ESP8266](#beispiele) Beispiel so, dass Ihre Webseite geholt wird.

<details><summary>Lösung</summary>

Folgende Zeile in WLAN Modem ESP8266 lowlevel Beispiel ändern

    send( (char *) "AT+CIPSTART=4,\"TCP\",\"httpbin.org\",80" );  // Socket zu google.com oeffnen

</p></details> 

### ATCmdParser

Verwenden Sie den [ATCmdParser](https://os.mbed.com/docs/mbed-os/latest/apis/atcmdparser.html) von mbed um eine Webseite zu holen.<br>

* Anwendungen: Kommunikation mit AT Befehlsatz kompatiblen Modems. 

<details><summary>Lösung</summary>
  
    /* ATCmdParser usage example
     */
    
    #include "mbed.h"
    #include "platform/ATCmdParser.h"
    
    #define   ESP8266_DEFAULT_BAUD_RATE   115200
    
    UnbufferedSerial *serial;
    ATCmdParser *parser;
    
    int main()
    {
        printf( "\nATCmdParser with ESP8266 example" );
    
        serial = new UnbufferedSerial( MBED_CONF_IOTKIT_ESP8266_TX, MBED_CONF_IOTKIT_ESP8266_RX );
        serial->baud( ESP8266_DEFAULT_BAUD_RATE );
        parser = new ATCmdParser( serial );
        parser->debug_on( 1 );
        parser->set_delimiter( "\r\n" );
    
        //Now get the FW version number of ESP8266 by sending an AT command
        printf( "\nATCmdParser: Retrieving FW version" );
        parser->send( "AT+GMR" );
        int version;
        if ( parser->recv( "SDK version:%d", &version ) && parser->recv( "OK" ) )
        {
            printf( "\nATCmdParser: FW version: %d", version );
            printf( "\nATCmdParser: Retrieving FW version success" );
        }
        else
        {
            printf( "\nATCmdParser: Retrieving FW version failed" );
            return -1;
        }
    
        parser->send( "AT+CWMODE=1" );  // Station Mode, d.h. Modem = Client zu AP
        if  ( parser->recv( "OK" ) )
        {
            parser->send( "AT+CIFSR" );  // Ausgabe IP-Adresse
            parser->recv( "OK" );
            parser->send( "AT+CIPMUX=1" );  // Mehrere Verbindungen aktivieren
            parser->recv( "OK" );
            parser->send( "AT+CIPSTART=4,\"TCP\",\"httpbin.org\",80" );  // Socket zu google.com oeffnen
            parser->recv( "OK" );
            parser->send( "AT+CIPSEND=4,18" );  // 11 Zeichen senden
            parser->recv( "OK" );
            parser->send( "GET / HTTP/1.0\r\n" );  // HTTP GET
            parser->recv( "OK" );
        }
    
        printf( "\nDone\n" );
    }
  
</p></details> 