import threading
import time
import psutil
import requests


CPU_THRESHOLD = 80  
MEMORY_THRESHOLD = 80   


send_timestamps = {}
error_logs = []  

class SendSMS:
    def __init__(self, phone_number, proxy=None):
        self.phone_number = phone_number
        self.proxy = proxy  

    def send_otp(self):
       
        try:
            response = requests.post(
                "https://api.example.com/send_sms",  
                json={"phone": self.phone_number, "message": "Your OTP is 123456"},
                proxies={"http": self.proxy, "https": self.proxy} if self.proxy else None
            )
            return 'sent successfully' in response.text
        except Exception as e:
            error_logs.append(f"Error sending SMS: {e}")  
            return False

class SubmitSMS:
    def submit_otp(self, trigger_id, sms_code):
       
        try:
            response = requests.post(
                "https://api.example.com/submit_otp",  
                json={"trigger_id": trigger_id, "sms_code": sms_code}
            )
            return 'submitted successfully' in response.text
        except Exception as e:
            error_logs.append(f"Error submitting OTP: {e}")  
            return False

def manage_sms_service(action, country, operator, program_name):
    
    container_name = f"sms_service_{country}_{operator}_{program_name}"
    if action == "start":
        print(f"Started SMS service for: {container_name}")
    elif action == "stop":
        print(f"Stopped SMS service for: {container_name}")

def rate_limit(country):
    """Ensure no more than 10 SMS are sent per minute for each country."""
    if country not in send_timestamps:
        send_timestamps[country] = []

    
    send_timestamps[country] = [
        t for t in send_timestamps[country] if time.time() - t < 60
    ]

    if len(send_timestamps[country]) < 10:
        send_timestamps[country].append(time.time())
        return True
    else:
        print(f"Rate limit reached for {country}. Please wait before sending more SMS.")
        return False

def monitor_system_health():
    """Monitor CPU and memory usage to prevent overload."""
    while True:
        cpu_usage = psutil.cpu_percent(interval=1)
        memory_info = psutil.virtual_memory()

        if cpu_usage > CPU_THRESHOLD:
            print(f"Alert! High CPU usage: {cpu_usage}%")

        if memory_info.percent > MEMORY_THRESHOLD:
            print(f"Alert! High Memory usage: {memory_info.percent}%")

        time.sleep(10)

def check_and_restart_sms_service(country, operator, program_name):
    
    container_name = f"sms_service_{country}_{operator}_{program_name}"
    print(f"Checking status of {container_name}... (Placeholder logic)")

def log_system_metrics():
    
    with open("system_metrics.log", "a") as f:
        while True:
            cpu_usage = psutil.cpu_percent(interval=1)
            memory_info = psutil.virtual_memory()
            f.write(f"CPU Usage: {cpu_usage}% | Memory Usage: {memory_info.percent}%\n")
            f.flush()
            time.sleep(10)  

def send_sms(country, operator, program_name):
    
    if rate_limit(country):
        phone_number = "1234567890"  
        proxy = None 
        
        sms_sender = SendSMS(phone_number, proxy)
        
        if sms_sender.send_otp():
            print(f"Sending SMS for {country} - {operator} via {program_name}")
            
            sms_code = "123456"  
            trigger_id = "example_trigger_id"              sms_submitter = SubmitSMS()
            if sms_submitter.submit_otp(trigger_id, sms_code):
                print("OTP submitted successfully.")
            else:
                print("Failed to submit OTP.")
        else:
            print("Failed to send SMS.")

def log_errors():
    """Log errors collected in error_logs at a controlled rate."""
    last_error_log_time = time.time()
    while True:
        if error_logs and (time.time() - last_error_log_time) > 30:  
            with open("error_log.txt", "a") as log_file:
                while error_logs:
                    log_file.write(error_logs.pop(0) + "\n")
            last_error_log_time = time.time()
        time.sleep(1)

if __name__ == "__main__":
   
    sms_services = [
        ("US", "Verizon", "Program1"),
        ("IN", "Airtel", "Program2"),
        ("UZ", "UzMobile", "Program3"),
    ]

   
    for country, operator, program_name in sms_services:
        manage_sms_service("start", country, operator, program_name)

   
    threading.Thread(target=monitor_system_health, daemon=True).start()
    threading.Thread(target=log_system_metrics, daemon=True).start()
    threading.Thread(target=log_errors, daemon=True).start()  

   
    try:
        while True:
            for country, operator, program_name in sms_services:
                send_sms(country, operator, program_name)
                check_and_restart_sms_service(country, operator, program_name)
            time.sleep(6)  
    except KeyboardInterrupt:
        print("Terminating the SMS service management.")
