1. Connectez-vous au [portail Classic](http://manage.windowsazure.com). 
2. Dans la barre de commandes en bas de la fenêtre, cliquez sur **Nouveau**.
3. Sous **Calculer**, cliquez sur **Machine virtuelle**, puis sur **À partir de la galerie**.
   
    ![Accéder à À partir de la galerie dans la barre de commandes](./media/virtual-machines-create-WindowsVM/fromgallery.png)
4. Le premier écran vous propose de **Choisir une image** pour votre machine virtuelle dans la liste d’images disponibles. Vous pouvez choisir une image dans la galerie ou effectuer votre choix parmi les images et les disques que vous avez chargés. Les images disponibles dépendent de l’abonnement que vous utilisez.
5. Le deuxième écran vous permet de choisir un nom d’ordinateur, la taille, un nom d’utilisateur et un mot de passe d’administration. Utilisez le niveau et la taille requis pour exécuter votre application ou charge de travail. Voici quelques conseils :
   
   * **nom de la machine virtuelle** peut contenir uniquement des lettres, des chiffres et des traits d'union. Il doit commencer par une lettre et se terminer par une lettre ou un chiffre.
   * **Nouveau nom d’utilisateur** fait référence au compte administratif que vous utilisez pour gérer le serveur. Le mot de passe doit compter 8 à 123 caractères et au moins trois des types de caractère suivants : un caractère minuscule, un caractère majuscule, un chiffre et un caractère spécial. **Vous aurez besoin du nom d'utilisateur et du mot de passe pour vous connecter et vous identifier sur la machine virtuelle**.
   * La taille d'une machine virtuelle affecte son coût d'utilisation, ainsi que les options de configuration comme le nombre de disques de données que vous pouvez joindre. Pour en savoir plus, voir la rubrique [Tailles de machines virtuelles](../articles/virtual-machines/virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
6. Le troisième écran vous permet de configurer les ressources pour la mise en réseau, le stockage et la disponibilité. Voici quelques conseils :
   
   * Le **Nom du cloud Service DNS** est le nom DNS global qui devient un élément de l'URI utilisé pour contacter la machine virtuelle. Vous devrez créer votre propre nom de service cloud, car il doit être unique dans Azure. Les services cloud sont importants pour les scénarios utilisant [plusieurs machines virtuelles](../articles/virtual-machines/virtual-machines-windows-classic-connect-vms.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).
   * Pour **Région/Groupe d'affinités/Réseau virtuel**, utilisez une région qui correspond au lieu où vous êtes. Vous pouvez également choisir de spécifier un réseau virtuel à la place.
   * Si vous voulez qu’une machine virtuelle utilise un réseau virtuel, lorsque vous la créez, vous **devez** indiquer le réseau virtuel. Vous ne pouvez pas joindre la machine virtuelle à un réseau virtuel après avoir créé celle-ci. Pour plus d’informations, consultez l’article [Vue d’ensemble de Réseau virtuel Azure](../articles/virtual-network/virtual-networks-overview.md).
   * Pour plus d’informations sur la configuration des points de terminaison, consultez l’article [Configuration des points de terminaison sur une machine virtuelle](../articles/virtual-machines/virtual-machines-windows-classic-setup-endpoints.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).
7. Le quatrième écran de configuration vous permet d’installer l’agent de machine virtuelle et de configurer certaines des extensions disponibles.
   
   > [!NOTE]
   > L’agent de machine virtuelle fournit l’environnement dans lequel vous installez les extensions qui permettent d’interagir avec la machine virtuelle ou de la gérer. Pour plus d’informations, consultez l’article [À propos de l’agent et des extensions de machine virtuelle](../articles/virtual-machines/virtual-machines-windows-classic-agents-and-extensions.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).  
   > 
   > 
8. Une fois que vous avez créé la machine virtuelle, la version classique du portail la répertorie sous **Machines virtuelles**. Le service cloud et le compte de stockage correspondants sont également créés et répertoriés dans ces sections. La machine virtuelle et le service cloud sont démarrés automatiquement et leur statut est répertorié comme **En cours d'exécution**.
   
    ![Configurer l’agent de machine virtuelle et les points de terminaison de la machine virtuelle](./media/virtual-machines-create-WindowsVM/vmcreated.png)



<!--HONumber=Nov16_HO3-->


