# Billiard-SRO-
Main Billiard yukk
#%%
from coppeliasim_zmqremoteapi_client import RemoteAPIClient
import math
import time
import keyboard

print("Program Started")

client = RemoteAPIClient('127.0.0.1', 23000)
sim = client.getObject('sim')

# Mulai simulasi
sim.startSimulation()

#%%
try:
    white_ball_handle = sim.getObject("/Sphere[6]")  
    print(f"White ball handle: {white_ball_handle}")
except:
    print("Error: Tidak dapat menemukan bola putih (Sphere[6])")
    sim.stopSimulation()
    exit()

initial_pos = sim.getObjectPosition(white_ball_handle, -1)
print(f"Initial position: {initial_pos}")

bounce_distance = 1.0  

key_pressed = {"w": False, "a": False, "s": False, "d": False}

print("\n=== KONTROL BOLA BILLIARD ===")
print("W: Pantul ke atas (Y+)")
print("S: Pantul ke bawah (Y-)")
print("A: Pantul ke kiri (X-)")
print("D: Pantul ke kanan (X+)")
print("R: Reset Posisi")
print("ESC: Keluar dari Program")
print("=============================")

def apply_bounce(direction):
    """Menerapkan pantulan ke bola dengan arah tertentu"""
    try:
        current_pos = sim.getObjectPosition(white_ball_handle, -1)
       
        if direction == "up":
            new_pos = [current_pos[0], current_pos[1] + bounce_distance, current_pos[2]]
        elif direction == "down":
            new_pos = [current_pos[0], current_pos[1] - bounce_distance, current_pos[2]]
        elif direction == "left":
            new_pos = [current_pos[0] - bounce_distance, current_pos[1], current_pos[2]]
        elif direction == "right":
            new_pos = [current_pos[0] + bounce_distance, current_pos[1], current_pos[2]]
   
        sim.setObjectPosition(white_ball_handle, -1, new_pos)
        print(f"Bola dipantulkan ke {direction}")
 
        steps = 10
        for i in range(steps):
            progress = (i + 1) / steps
            intermediate_pos = [
                current_pos[0] + (new_pos[0] - current_pos[0]) * progress,
                current_pos[1] + (new_pos[1] - current_pos[1]) * progress,
                current_pos[2]
            ]
            sim.setObjectPosition(white_ball_handle, -1, intermediate_pos)
            time.sleep(0.01)
            
    except Exception as e:
        print(f"Error menerapkan pantulan: {e}")

try:
    running = True
    while running:
        if keyboard.is_pressed('w') and not key_pressed["w"]:
            apply_bounce("up")
            key_pressed["w"] = True
        elif not keyboard.is_pressed('w'):
            key_pressed["w"] = False
            
        if keyboard.is_pressed('s') and not key_pressed["s"]:
            apply_bounce("down")
            key_pressed["s"] = True
        elif not keyboard.is_pressed('s'):
            key_pressed["s"] = False
            
        if keyboard.is_pressed('a') and not key_pressed["a"]:
            apply_bounce("left")
            key_pressed["a"] = True
        elif not keyboard.is_pressed('a'):
            key_pressed["a"] = False
            
        if keyboard.is_pressed('d') and not key_pressed["d"]:
            apply_bounce("right")
            key_pressed["d"] = True
        elif not keyboard.is_pressed('d'):
            key_pressed["d"] = False
            
        if keyboard.is_pressed('r'):  
            sim.setObjectPosition(white_ball_handle, -1, initial_pos)
            print("Posisi direset ke awal")
            time.sleep(0.5)  
            
        if keyboard.is_pressed('esc'): 
            print("Keluar dari program")
            running = False
            
        time.sleep(0.01)  

except Exception as e:
    print(f"Error: {e}")

finally:
    sim.stopSimulation()
    print("Simulasi dihentikan")
