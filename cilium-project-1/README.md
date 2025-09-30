# Cilium-project-1

## benodigdheden

* Helm
* Een standaard kubernetes cluster. Standaard POD/service CIDR, geen BPF als kube-proxy replacement etc.
* Een router die BGP ondersteunt (Als alternatief kan het Bird softwarepackage gebruikt worden om je BGP sessies mee op te opzetten.)

## Te volgen stappen

### Zorg er voor dat Kubernetes worker nodes waarop BGP peers actief zijn het juiste label krijgen.
1. kubectl label nodes worker1 worker2 worker3 bgp=jazeker --overwrite

### Zorg er voor dat de Helm repo toegevoegd is:
2. helm repo add cilium https://helm.cilium.io/

### Update de Helm repo:
3. helm repo update

### Installeer Cilium:
4. helm upgrade --install cilium cilium/cilium --version 1.18.2 --namespace cilium-cni --create-namespace -f values.yaml

# Controleer of de pods goed starten
5. kubectl -n cilium-cni get pods

### Controleer of de BGP gerelateerde CRD's van Cilium toegevoegd zijn:
6. kubectl get crd|grep bgp

### Controleer of de Kubernetes nodes de status "Ready" hebben.
7. kubectl get nodes

### Bekijk de yaml file met de BGP gerelateerde instellingen.
8. Bekijk de ASnummers
   Bekijk het IP adres van de peer. Deze moet in jouw geval mogelijk aanegpast worden naar jouw situatie.
   let op dat bij CiliumBGPADvertisement LoadBalancerIP aanstaat. Dat zijn service IP's van het type loadbalancer. Deze willen we via BGP routeren    en dus van buiten het cluster bereikbaar maken.
   Kijk bij CiliumLoadBalancerIPPool naar de range die bij CIDR is opgegeven. Dit is in jouw geval mogelijk ook anders.
   Let op matchLabels: Alleen services die een matchend label hebben en van type Loadbalancer zijn, worden gerouteerd via BGP.

### Installeer de eventueel aangepaste CRD's
9. kubectl kubectl -n cilium-cni apply -f bgppeerconfig.yaml


### Configureer de BGP peer (Externe router)
10. Maak per worker een BGP connection aan. let op. de connections moeten NIET op connect staan. Cilium heeft geen listener op de BGP poort hier. 
    De Connections moeten wel op Listen staan. Verder het RouterID ingevuld worden. Uiteraard moeten de ASnummers overeenkomen met die van de CRD's

### Debuggen
11. Gebruik de cilium binary. "cilium -n cilium-cni bgp peers" laat de geconfigureerde peers zien. "cilium -n cilium-cni bgp routes" laat de geadverteerde routes zien.

## eindresultaat
Wat heb je nu? Een basis Kubernetes cluster dat qua CNI configuratie een goede basis vormt. Cilium kan veel en uiteraard is dit het topje van de ijsberg. Als dit werkt, heb je in ieder geval een Kubernetes cluster met goed werkende CNI waarmee je eenvoudig bijvoorbeeld de ingress controller van buiten het cluster bereikbaar kan maken. Een goede setup dus om je bijvoorbeeld  mee voor te bereiden op certificeringen.  





