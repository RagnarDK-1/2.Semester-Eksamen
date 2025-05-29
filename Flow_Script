import pigpio #importere pigpio, pigpio er et library som hjælper med at kontrollere GPIo'er, fungere med alle versioner af Pi
import mariadb #importere mariadb library, som hjælper scriptet med at forbinde til den rigtige maria database
import time #tids library, hjælper med at tælle i sekunder, kan også bruge dato og tid moduler

# --- Flowmåler konfiguration ---
FLOW_PIN = 18 #Definere flowmåler puls tæller til pin 18
PULSES_PER_LITER = 450 # 450 Pulser er 1 liter,
pulse_count = 0 # Tæller hvor mange pulser den modtager, starter på 0
liter_total = 0.0 # Holder styr på hvor mange liter der er målt, starter også på 0

# --- Pulsregistrering ---
def pulse_callback(gpio, level, tick): #Bliver kaldt hvis der kommer en ædnring i flowmåleren
    global pulse_count #Global, gør det muligt at ændre pulse_count da det er en variabel
    if level == 0:  # faldende kant, hjælper med at tælle pulser, falder den fra 1 -> 0 tæller den 1 puls som fortæller mængden af vandgennemstrømning        pulse_count += 1 #Øger puls tælleren med 1
        pulse_count += 1 # Øger puls tælleren med 1
# --- Opsæt pigpio ---
pi = pigpio.pi() #Opretter forbindelse til pigpio, som gør det muligt at styre GPIO pins
if not pi.connected: #Tjekker forbindelsen lykkedes eller ikke gør, via software, om pigpio er aktiveret, ellers "sudo pigpiod"
    print("Kunne ikke forbinde til pigpio") #Fortæller at problemet er i pigpio og ikked andre steder
    exit() #Stopper programmet, da vi ikke kan fortsætte uden pigpio

#Elektrisk opsætning, hvordan skal pi'en opføre sig når der kommer pulser.
pi.set_mode(FLOW_PIN, pigpio.INPUT) #Vi har ikke fortalt hvad pin 18 bruges til, her fortæller vi at det er input
pi.set_pull_up_down(FLOW_PIN, pigpio.PUD_UP) #Aktivere en modstand(pull-up) så den ikke modtagere tilfældige signaler men et klart signal når den bru>cb = pi.callback(FLOW_PIN, pigpio.FALLING_EDGE, pulse_callback)#hver gang der kommer en puls, vil callback kalde efter denne funktion
#uden den her linje sker der intet når flowmåleren sender et signal


def opdater_database(liter):  #når ny måling af liter bliver detekteret vil den forsøge at opdatere
    #liter er argumentet af antallet målt siden sidste opdatering og gemmer derefter den nye "liter" funktion
    print(f"Forbinder til database for at opdatere {liter:.3f} liter.")
    try:
        conn = mariadb.connect( 
            user="admin",
            password="admin",
            host="152.115.77.165",
            port=50110,
            database="ølbord",
            connect_timeout=3
        )
        cur = conn.cursor() #Cursor bruges til at skrive i en SQL kommando, conn.cursor er så forbindelsen til SQL og og sende kommandoer
        cur.execute(
            "UPDATE drikke_logg SET liter_drukket = liter_drukket + ? WHERE id =1",
            (liter,)
        )
        #Lægger nye mængde liter til den eksiterende tal i 'liter_drukket'
        conn.commit()
        #Gemmer ændring i DB
        conn.close()
        #Lukker forbindelsen til DB
        print(f"Database opdateret med {liter:.3f} liter")
    except mariadb.Error as e:
        print(f"Fejl ved databaseopdatering: {e}")

# --- Hovedløkke ---
def main_loop(): #Kører hele tiden og måler vandforbrug
    global pulse_count, liter_total #Gør det muligt at læse og ændre variablerne,
    print("Starter måling. Ctrl+C for at stoppe.")
    cb = pi.callback(FLOW_PIN, pigpio.FALLING_EDGE, pulse_callback) #Holder øje med pinnen hvis der sker noget, og så "callbacker" den når der kommer en puls, og bliver kørt igen, cb gør at literen bliver gemt i en vairablel ved cb.cancel 
    try:
        while True:
            start = pulse_count #Gemmer antal pulser ved starten af intervallet
            time.sleep(1) #Måle intervallet, hvor lang tid skal vi måle pulser
            end = pulse_count #Gemmer antallet af pulser efter i dette tilfælde 1 sekund
            pulses = end - start # Udregner hvor mange pulser der er kommet mellem slutningen og starten
            liter = pulses / PULSES_PER_LITER # Konvertere pulser til liter
            liter_total += liter #Ligger literne til det samlede forbrug

            # Vis terminalstatus uanset hvad
            print(f" {pulses} pulser => {liter:.3f} liter         I alt: {liter_total:.3f} liter")
            #Hvis der bliver registreret 1 puls eller mere vil den opdater database liter eller
 if pulses > 0:
                opdater_database(liter)

    except KeyboardInterrupt:
        print("\n program Stoppet af bruger.")
    finally:
        cb.cancel() #Stopper pigpio callback
        pi.stop() #Lukker pigpio forbindelse

if __name__ == "__main__": #hvis koden bliver kørt direkte skal den kalde på main_loop
    #Hvis filen bliver startet i et andet program skal den bare stille funktionerne til rådighed
    main_loop()
