# Maven Sonatype Sample

Ce projet Java sert d'exemple simple pour le déploiement d'un jar sur **Sonatype** à l'aide de **Maven**.

## Générer la clé GPG

La génération d'une clé **GPG** est nécessaire pour signer le jar avant de le publier sur **Sonatype**. Pour cela, exécutez les commandes suivantes :

1. Exécutez `gpg --full-generate-key` et répondez aux questions afin de générer votre clé GPG.
2. Exécutez `gpg --list-keys` afin de récupérer le **KEY_ID** correspondant à la clé générée.
3. Exécutez `gpg --keyserver keyserver.ubuntu.com --send-keys <KEY_ID>` pour publier la clé publique sur le serveur de clés.
4. Exécutez `gpg --armor --export-secret-keys <KEY_ID>` pour récupérer votre clé privée. Celle-ci sera nécessaire dans GitHub.

Attention, vous pouvez partager la clé publique sans risque, mais vous ne devez jamais partager la clé privée.

## Créer un compte Sonatype

Maintenant que la clé GPG est générée et publiée, créez votre compte **Sonatype** sur le [site officiel](https://central.sonatype.com/) en cliquant sur le bouton **Sign Up**.

## Créer le namespace

À présent, vous devez créer un namespace pour votre projet en suivant les étapes ci-dessous.

1. Ouvrez la page [namespaces](https://central.sonatype.com/publishing/namespaces).
2. Saisissez le namespace que vous souhaitez créer, par exemple **fr.damnardev** si votre nom de domaine est **damnardev.fr**.
3. Validez que vous êtes bien propriétaire du namespace en suivant les instructions fournies par **Sonatype**. Actuellement, cela consiste à ajouter un enregistrement **TXT** à votre DNS pour prouver que vous possédez le domaine correspondant au namespace.

## Générer le token

Il est maintenant temps de générer le token **Sonatype** afin de l'utiliser depuis **Maven**. Pour cela, suivez les étapes ci-dessous.

1. Ouvrez la page [user token](https://central.sonatype.com/usertoken) et cliquez sur **Generate User Token**.
   - Renseignez un nom pour votre token.
   - Renseignez une date d'expiration.
   - Cliquez sur **Create Token**.
2. Sauvegardez en lieu sûr les informations des sections **Use the following in your Maven settings.xml**, **Username**, **Password** et **Username : Password (base64)**.

## Configurer Maven

Il est maintenant temps de configurer **Maven**. Ouvrez le fichier **~/.m2/settings.xml** et ajoutez-y les informations de votre token générées lors de l'étape **Générer le token** ainsi que les détails de votre clé GPG.

Dans la section `servers`, ajoutez les informations ci-dessous:

```xml
<servers>
	<server>
		<id>central</id>
		<username>SONATYPE_USERNAME</username>
		<password>SONATYPE_PASSWORD</password>
		<repositoryOrigins>
			<repositoryOrigin>https://central.sonatype.com</repositoryOrigin>
		</repositoryOrigins>
	</server>
</servers>
```

Puis, dans la section `profiles`, ajoutez le **KEY_ID** de votre clé GPG :

```xml
<profile>
	<id>gpg</id>
	<properties>
		<gpg.keyname>KEY_ID</gpg.keyname>
	</properties>
</profile>
```

Enfin, créez une variable d'environnement **MAVEN_GPG_PASSPHRASE** contenant la passphrase de votre clé GPG.

## Tester localement

Vous pouvez tester la publication de vos artefacts depuis votre poste en exécutant la commande suivante :

```shell
./mvnw clean deploy -Psonatype,gpg
```

Cette commande va :

1. Compiler votre projet.
2. Générer les sources et la documentation **Javadoc**.
3. Signer les artefacts avec votre clé GPG en activant le profil **gpg**, option `-Pgpg`.
4. Déployer les artefacts dans le dépôt **Sonatype** en activant le profil **sonatype**, option `-Psonatype`.

Les profils **sonatype** et **gpg** doivent obligatoirement être spécifiés dans la commande, sinon la publication ne fonctionnera pas.

Si vous spécifiez le profil **sonatype** sans le profil **gpg**, la publication échouera, car les artefacts ne seront pas signés avec votre clé GPG.

## GitHub

Pour publier les artefacts sur **Sonatype**, vous pouvez utiliser la GitHub Action **publish**, qui se lance manuellement ou après la création d'une pull request ou d'un merge sur la branche **develop**. Avant de l'exécuter, vous devez configurer les secrets suivants dans votre dépôt GitHub :

- **SONATYPE_USERNAME** : le **username** généré durant l'étape **Générer le token**
- **SONATYPE_PASSWORD** : le **password** généré durant l'étape **Générer le token**
- **GPG_PRIVATE_KEY** : votre clé privée GPG exportée avec `gpg --armor --export-secret-keys <KEY_ID>`
- **GPG_PASSPHRASE** : la passphrase de votre clé GPG
