# -----------------------------------------------------------
#   Script that allows sending commands to multiple network devices concurrently using the Netmiko library and Python semaphore
#   
#   Netmiko https://github.com/ktbyers/netmiko © Kirk Byers
#   2021 © https://github.com/alexfour
# -----------------------------------------------------------

from netmiko import ConnectHandler
from getpass import getpass, getuser
from multiprocessing import Process
from multiprocessing import Semaphore
import time

def send_commands(device_dict,sema):
    """
    Summary:
    Connects to the device via SSH and executes commands.
    After execution closes the connection and releases the process

    Arguments:
    device_dict - Dictionary containing information from a device
    sema        - Semaphore object

    """
    print('Processing {}'.format(device_dict['host']))

    #Establish connection to the device
    net_connect = ConnectHandler(**device_dict)

    #Use Netmiko to save devices current running configuration
    output = net_connect.save_config() 

    #Send 'show flash' command and save output to logfile
    output += net_connect.send_command(
        command_string="show flash",
        strip_prompt=False,
        strip_command=False
    )
        
    #Close connection
    net_connect.disconnect()

    #Release this process
    sema.release()
    print('Process {} is done'.format(device_dict['host']))

if __name__ == '__main__':
    """
    Main function where the following steps are performed:
    - Getting user credentials for SSH (g_user, g_password)
    - Creating objects for each IP-address in 'addresses.txt'
    - Tracking how long execution takes
    - Using semaphore to create multiple concurrent processes

    """
    #Get username and password
    g_user = getuser()
    g_password = getpass()

    #Track how long execution takes
    start_time = time.perf_counter()

    #Default device template
    hp_template = { 
        "device_type": "hp_procurve",
        "host": "",
        "username": g_user,
        "password": g_password,
        "session_log": ""  
    }

    #Get IP-addresses from addresses.txt
    IP_List = open('addresses.txt').read().splitlines()

    #Setup device_list to contain devices
    device_list = []

    #Create new device for each IP
    for IP in IP_List:
        new_device = hp_template.copy()
        #Edit variables for each new device
        new_device['host'] = IP
        new_device['session_log'] = "%s_output.txt"%IP  #Set session logfile filename
        device_list.append(new_device)

    concurrency = 5     #Set the amount of concurrent tasks
    total_task_num = len(device_list)   #Calculate total amount of devices
    sema = Semaphore(concurrency)
    all_processes = []
    for i in range(total_task_num):
        sema.acquire()
        p = Process(target=send_commands, args=(device_list[0], sema))  #Create processes
        del device_list[0]
        all_processes.append(p)
        p.start()  
    
    #Wait for processes to finish
    for p in all_processes:
        p.join()

    finish_time = time.perf_counter()
    print (f'Finished %s processess in {round(finish_time-start_time,2)} second(s)'%total_task_num)
