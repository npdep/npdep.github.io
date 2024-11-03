# Architecture

[Start](../README.md)

- [Architecture](#architecture)
  - [Scenario](#scenario)
  - [Capabilities](#capabilities)
  - [Components](#components)

## Scenario

Imagine a scenario at a company, where sensitive data – financial reports, client information, and internal strategies – is carefully stored behind layers of security protocols. The IT team has fortified the network with firewalls, intrusion detection systems, and strict access controls. However, unknown to them, a threat actor has infiltrated the network using a phishing email, gaining access to a low-privileged account. From there, they begin the slow process of data exfiltration, carefully evading detection by blending their traffic with legitimate network activities.

The attacker has a methodical plan: rather than transferring large volumes of data in one go, they exfiltrate small bits of data over time, using unconventional channels like DNS tunneling, covert HTTP headers, and encrypted payloads in seemingly innocuous traffic. As these transmissions mimic normal network behavior, the data slips under the radar of conventional detection systems. Each packet of data holds a fragment of valuable information, which, over days and weeks, accumulates in the attacker’s external server – a treasure trove of sensitive data gathered without raising suspicion.

## Capabilities

From this scenario, we can derive the following core capabilities of data exfiltration via network protocol:

1. **Identify:**
    After successfully compromising a system, the attacker wants to identify relevant data on the system, or in the network, the compromised system is connected to.

2. **Collect:**
    Once the attacker has identified relevant data, it must be collected, especially if it is stored on a system in the network.

3. **Transfer:**
   After successful collection, the data is transferred via network protocol to a system controlled by the attacker outside the compromised network.

4. **Receive:**
   On the attacker side, the incoming data must be received.

5. **Extract:**
   As the exfiltrated data can be transported in different ways in the respective protocols depending on the scenario, this data must be extracted by the attacker.

6. **Save:**
   Since a file to be exfiltrated is usually transferred in small chunks, the attacker must save the file correctly from these chunks after extraction in order to make the original file readable for him.


## Components

To enable these capabilities in order to simulate real world scenarios, NPDEP is divided into two components:

- **npdep-client**
  The client is deployed on the compromised system and enables the identification, collection and transmission of the data to be exfiltrated.

- **npdep-server**
  The server is provided on a system controlled by the attacker outside the compromised network and enables the data to be received, extracted and saved.
  
For more information please have a look at the following topics:

- [Client](./client.md)
- [Server](./server.md)