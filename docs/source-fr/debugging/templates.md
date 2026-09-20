# Débogage des modèles

Vous pouvez déboguer les modèles UIX Jinja2 en plaçant le commentaire `{# uix.debug #}` n'importe où dans votre modèle. Vous verrez les messages de débogage lors de la liaison du modèle, de la mise à jour de valeur, de la réutilisation, de la déliaison et du désabonnement final. Chaque modèle reste abonné dans le cache pendant 20 secondes afin de faciliter l'application des modèles et d'améliorer légèrement la vitesse lorsque vous passez d'une vue à l'autre ou utilisez le même modèle sur des cartes de vues différentes.
