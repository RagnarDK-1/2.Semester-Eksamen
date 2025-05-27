import RPi.GPIO as GPIO          # Importerer biblioteket til at styre Raspberry Pi's GPIO-pins
import socket                    # Importerer biblioteket til netværksforbindelser (bruges til at teste internet)
import mariadb                  # Importerer biblioteket til at oprette forbindelse til en MariaDB-database
import time                      # Importerer biblioteket til at bruge pauser og tid

# --- GPIO opsætning ---
INTERNET_LED = 17                # Definerer GPIO pin 17 til at vise internetstatus med en LED
DB_LED = 27                      # Definerer GPIO pin 27 til at vise databasestatus med en LED

GPIO.setmode(GPIO.BCM)           # Angiver at vi bruger GPIO-nummerering (ikke fysisk pin-nummerering)
GPIO.setup(INTERNET_LED, GPIO.OUT)  # Sætter INTERNET_LED-pinen som output (LED'en kan tændes og slukkes)
GPIO.setup(DB_LED, GPIO.OUT)         # Sætter DB_LED-pinen som output (LED'en kan tændes og slukkes)

# --- Database konfiguration ---
db_config = {
    "user": "admin",             # Brugernavn til databasen
    "password": "admin",         # Adgangskode til databasen
    "host": "152.115.77.165",    # IP-adressen på databasen
    "port": 50110,               # Portnummer som databasen lytter på
    "database": "ølbord",        # Navn på databasen
    "connect_timeout": 3         # Timeout i sekunder hvis der ikke kan oprettes forbindelse
}

# --- Tjek internetforbindelse ---
def har_internet(host="8.8.8.8", port=53, timeout=2):  # Funktion der forsøger at forbinde til Googles DNS-server (på port 53) for at tjekke om der er internet
    try:
        socket.setdefaulttimeout(timeout)              # Sætter timeout til fx 2 sekunder – så den ikke venter for længe
        socket.socket(socket.AF_INET, socket.SOCK_STREAM).connect((host, port))  # Opretter en TCP-forbindelse til DNS-serveren – hvis den lykkes, er der internet
        return True                                    # Hvis forbindelsen lykkes, returner True (internet OK)
    except socket.error:                               # Hvis forbindelsen fejler, fanges fejlen her
        return False                                   # Returner False (ingen internetforbindelse)

# --- Tjek databaseforbindelse ---
def har_database():                                    # Funktion der forsøger at oprette forbindelse til databasen
    try:
        conn = mariadb.connect(**db_config)            # Forsøger at oprette forbindelse med de angivne databaseindstillinger
        conn.close()                                   # Lukker forbindelsen med det samme igen – vi tester kun forbindelsen
        return True                                    # Hvis forbindelsen lykkes, returner True
    except mariadb.Error:                              # Hvis noget går galt (forkert IP, database nede, forkert login osv.)
        return False                                   # Returner False (databasen kan ikke nås)

# --- Hovedprogram ---
try:
    print("Starter LED-overvågning af internet og database")  # Udskriver status til terminalen
    while True:                                   # Uendelig løkke der kører hele tiden (indtil du trykker Ctrl+C)
        if har_internet():                        # Tjekker om der er internet
            GPIO.output(INTERNET_LED, GPIO.HIGH)  # Tænder internet-LED'en hvis der er forbindelse
            print("Internetforbindelse oprettet")              # Udskriver status
        else:
            GPIO.output(INTERNET_LED, GPIO.LOW)   # Slukker internet-LED'en hvis forbindelsen fejler
            print("Internetforbindelse ikke oprettet")            # Udskriver status

        if har_database():                        # Tjekker om der er forbindelse til databasen
            GPIO.output(DB_LED, GPIO.HIGH)        # Tænder database-LED'en hvis der er forbindelse
            print("Databasefrobindelse oprettet")             # Udskriver status
        else:
            GPIO.output(DB_LED, GPIO.LOW)         # Slukker database-LED'en hvis der ikke er forbindelse
            print("Databaseforbindelse fejlet")           # Udskriver status

        time.sleep(5)                             # Venter 5 sekunder før den tjekker igen

except KeyboardInterrupt:                         # Hvis brugeren trykker Ctrl+C i terminalen
    print("Program stoppet af bruger.")         # Udskriver at programmet stoppes manuelt

finally:
    GPIO.output(INTERNET_LED, GPIO.LOW)           # Slukker internet-LED'en som oprydning
    GPIO.output(DB_LED, GPIO.LOW)                 # Slukker database-LED'en som oprydning
    GPIO.cleanup()                                # Rydder op i GPIO-indstillingerne, så pins ikke bliver hængende i output-tilstand
