# JENKINS SETUP #

## Jenkins UI:

- WebURL: http://localhost:8081/
- User: admin
- Password: konteineri käivitamise järgselt käsuga `docker compose -f docker-compose.yml up -d --build --force-recreate jenkins` käivita käsk `docker logs jenkins -f`, mis lubab vaadelda teenuse logi. Logisse tekib admin kasutaja jaoks genereeritud parool, mille abil saad keskkonda sisse.
