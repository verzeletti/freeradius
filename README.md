# CONSIDERAÇÕES INICIAIS

1. A instalação do freeradius foi realizada no momento da instalação do servidor Samba-AD, conforme [projeto Samba-AD](https://git.ifsc.edu.br/ctic/cte/samba4/samba4-ad "Samba-AD IFSC") coordenado pelo Igor

2. A informação da "Base DN" pode ser encontrada em "mods-enabled/ldap"

3. Realize um snapshot do servidor antes de iniciar (sempre é bom lembrar:)

4. Na árvore do Active Directory foi criada uma nova OU com o nome Campus_LAN (Ex. Urupema_LAN)

5. No arquivo "/root/sincronia/conexoes.conf" foi criada uma excessão de sincronismo, adicionando a linha "OUS_IGNORADOS="Urupema_LAN"

6. Recomendo a seguinte estrutura de OUs:

> ![mstsc_GCEFTc1rY4](https://github.com/verzeletti/freeradius/assets/23221957/c68d673b-3c3e-4e1a-a14b-4a72e77a3def)


7. Ao realizar a implementação, recomenda-se ativar o modo debug, adicionando o parâmetro "-X" no arquivo "/etc/default/freeradius".

8. O serviço pode ser reiniciado e monitorado pelo comando "systemctl restart freeradius.service ; tail -f /var/log/syslog", enquanto o modo debug estiver ativo

9. Crie o diretório "/etc/ssl/IFSC/" e copie para ele os certificados emitidos para o campus ou para a wifi (Ex: wifi.ifsc.edu.br.crt  e wifi.ifsc.edu.br.key)

10. Observe a estrutura da policy criada. Ela sugere um ID diferente para cada situação (ex: ID 120 para os "Domain Admins"), bem como a negação de acesso para um usuário no grupo "WIFI-Deny"



# IMPLEMENTAÇÃO

1. Edite o arquivo "clients.conf" e personalize ele com os IPs do dispositivos que farão uso do radius (ex: switch, fortigate, etc) e uma senha exclusiva para cada. Ou, siga o exemplo e deixe "*" no "ipv4addr" para usar a mesma senha em vários dispositivos (não recomendado)

2. Edite o arquivo "users" e comente as linhas relacionadas à atribuição de VLANs

3. Faça download do script "policy.d/vlan_ifsc" e altere os parâmetros das variáveis, logo no início, e os IDs de VLAN para cada caso testado pelo "if". Não esqueça de mudar o dono para "freerad" (ex: chown freerad:freerad policy.d/vlan_ifsc)

4. Edite o arquivo "mods-available/eap" e corrija os parâmetros "private_key_file" e "certificate_file"

5. Edite os arquivos "sites-available/default" e "sites-available/inner-tunnel" adicionado a opção "rewrite.vlan_ifsc" na seção "post-auth". 
    5.1 Aqui um detalhe para ficar atento! Se quiser restringir o radius para autenticar somente a wifi, pode fazer uso do laço "&Called-Station-Id && &Called-Station-Id =~ /.*:IFSC" na seção "authorize". Seguir modelos já existentes. Recomendo deixar comentado para que se possa fazer uso da autenticação 802.1X também nos switchs e opção de teste de Usuário Radius através do FortiGate.
    5.2 O servidor está projetado para autenticar automaticamente computadores no domínio, desde que estejam dentra da "OU Wifi" e em seu respectivo local. Note que, cada OU (ex: ou=Administrativos,ou=Wifi,...) corresponde à um ID de VLAN. E, não será solicitado usuário e senha para o usuário!

7. O uso de somente um SSID é recomendado e facilitará a vida de todo mundo (usuários e administradores de rede :)
    7.1 No fortiGate crie o SSID IFSC como "WPA3 Enterprise Transition", Optional VLAN ID = 0 e ative a opção "Dynamic VLAN assignment"  

6. Último detalhe, mais não menos importante: não esqueça de configurar os IDs de VLAN utilizados em todas as portas de switches que tem um Access Point conectado.
