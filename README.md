import ipaddress
from enum import Enum, auto

# --- IPCategory ENUM ---

class IPCategory(Enum):
    """
    Categorization of an IP address based on its usage, 
    ordered by check precedence.
    """
    INVALID = auto()
    LOOPBACK = auto()
    UNSPECIFIED = auto()
    LINK_LOCAL = auto()
    PRIVATE = auto()
    MULTICAST = auto()
    DOCUMENTATION = auto()
    RESERVED = auto()
    PUBLIC = auto()

# --- Core Classification Function ---

def classify_ip_address(ip_str: str) -> IPCategory:
    """
    Classifies a given IP address string into a specific usage category.
    """
    try:
        ip = ipaddress.ip_address(ip_str)
    except ValueError:
        return IPCategory.INVALID
    except Exception:
        return IPCategory.INVALID

    # 1. SPECIAL ADDRESSES
    if ip.is_loopback:
        return IPCategory.LOOPBACK
    
    if ip.is_unspecified:
        return IPCategory.UNSPECIFIED

    if ip.is_link_local:
        return IPCategory.LINK_LOCAL

    # 2. STANDARD IANA CLASSIFICATIONS
    if ip.is_private:
        return IPCategory.PRIVATE

    if ip.is_multicast:
        return IPCategory.MULTICAST

    # 3. DOCUMENTATION / TESTING (RFC 5737 and RFC 3849)
    if ip.version == 4:
        if ip in ipaddress.ip_network('192.0.2.0/24') or \
           ip in ipaddress.ip_network('198.51.100.0/24') or \
           ip in ipaddress.ip_network('203.0.113.0/24'):
            return IPCategory.DOCUMENTATION
    elif ip.version == 6:
        if ip in ipaddress.ip_network('2001:DB8::/32'):
            return IPCategory.DOCUMENTATION

    # 4. RESERVED
    if ip.is_reserved:
        return IPCategory.RESERVED

    # 5. DEFAULT
    return IPCategory.PUBLIC

# --- Continuous Execution (Main Program) ---

def run_interactive_mode():
    """Continuously prompts the user for IP addresses until terminated."""
    
    print("\n" + "="*50)
    print("      🌐 IP Address Category Detector (Interactive Mode)")
    print("="*50)
    print("Instructions: Enter an IPv4 or IPv6 address.")
    print("              Type 'exit' or 'quit' to terminate the program.")
    print("-" * 50)
    
    while True:
        try:
            # Note: In Colab, the 'input' prompt appears below the cell output area.
            ip_input = input("Enter IP: ").strip()
        except KeyboardInterrupt:
            # This handles kernel interruption, but standard 'exit' is cleaner.
            print("\n\nSession aborted (Kernel interrupted).")
            break

        cleaned_input = ip_input.lower()

        # Termination check
        if cleaned_input in ["exit", "quit", "q"]:
            print("\n👋 Thank you for using the IP Categorizer. Goodbye!")
            break
        
        if not cleaned_input:
            print("❗ Input cannot be empty. Please try again.")
            continue

        # Classify and display the result
        category = classify_ip_address(cleaned_input)
        
        # Display the result clearly
        print(f"  -> Input: {cleaned_input}")
        print(f"  -> Category: **{category.name}**\n")

# --- Program Entry Point ---

if __name__ == "__main__":
    run_interactive_mode()
