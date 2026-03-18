# iptables - Firewall & Traffic Management

Core tool for managing Linux kernel firewall rules: filtering traffic,
redirecting ports, and masquerading connections.

## 1. Viewing Current Rules
* `sudo iptables -L -n -v` - List all filter rules with packet counters.
* `sudo iptables -t nat -L -n -v` - List NAT rules (port forwarding, masquerade).
* `sudo iptables -L -n --line-numbers` - Show rules with line numbers (needed for deletion).

## 2. Port Forwarding (NAT Redirect)
* `sudo iptables -t nat -A OUTPUT -p tcp --dport <from> -j REDIRECT --to-port <to>` - Redirect local traffic (curl, local apps) from one port to another.
* `sudo iptables -t nat -A PREROUTING -p tcp --dport <from> -j REDIRECT --to-port <to>` - Redirect inbound external traffic from one port to another.

**Note:** To cover both local and external traffic, apply both rules simultaneously.
`PREROUTING` handles external inbound traffic only. `OUTPUT` handles traffic
generated on the same host.

## 3. Deleting Rules
* `sudo iptables -t nat -D OUTPUT <line_number>` - Delete a specific NAT OUTPUT rule by line number.
* `sudo iptables -t nat -F` - **Caution:** Flush (delete) all rules in the NAT table.
* `sudo iptables -F` - **Caution:** Flush all filter rules.

## 4. Persisting Rules After Reboot
* `sudo apt install iptables-persistent` - Install persistence package (Debian/Ubuntu).
* `sudo netfilter-persistent save` - Save current rules to disk.
* `sudo netfilter-persistent reload` - Reload saved rules.

## 5. Common Patterns
* Allow incoming traffic on a port:
  `sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT`
* Block traffic from a specific IP:
  `sudo iptables -A INPUT -s <IP> -j DROP`
* Allow established connections (stateful firewall baseline):
  `sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`