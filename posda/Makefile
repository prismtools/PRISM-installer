# a method of running an arbitrary script before anything else happens
# CHEAT_ARG := $(shell myscript)

.PHONY: remove nothing

# Set this to kubectl if you don't have oc
OC := kubectl
CMD_DEL := ${OC} delete --ignore-not-found

default: nothing

nothing:
	@echo Not doing anything
	@echo Try deploy or remove


deploy: pvcs.last configs.last redis/statefulset.last database.last posda-api/deployment.last posda/deployment.last ffp/deployment.last web/deployment.last worker/low.last worker/high.last kaleidoscope/deployment.last magicka/deployment.last

remove: remove-posda remove-posda-api remove-ffp remove-worker-low remove-worker-high remove-kaleidoscope remove-magicka remove-web remove-database remove-redis remove-database-secret remove-configs

# -------------- special targets ---------------------------- #

restart-web: 
	${OC} scale --replicas=0 deployment/web
	${OC} scale --replicas=1 deployment/web

restart-posda-things: 
	${OC} scale --replicas=0 deployment/posda deployment/worker-high deployment/worker-low
	${OC} scale --replicas=1 deployment/posda deployment/worker-high deployment/worker-low

# ----------------------------------------------------------- #

%.last: %.yaml
	${OC} apply -f $<
	touch $@

# some grouped things

pvcs.last: pvcs/posda-cache-auto-pvc.last pvcs/posda-postgres-pvc.last
database.last: database/statefulset.last database/secret.last
configs.last: configs/posda-api.last configs/common.last configs/posda-database.last configs/exodus.last configs/posda-nbia.last configs/posda-map.last configs/ream.last 

# remove rules ----------------------------------------------------------------

# TODO: redis needs a pvc!
remove-pvcs: remove-pvc-posda-cache remove-database-pvc
	rm -f pvcs/*.last

remove-configs:
	${CMD_DEL} configmaps/posda-api
	${CMD_DEL} configmaps/common
	${CMD_DEL} configmaps/posda-database
	${CMD_DEL} configmaps/exodus
	${CMD_DEL} configmaps/posda-nbia
	${CMD_DEL} configmaps/posda-map
	${CMD_DEL} configmaps/ream
	rm -f configs.last
	rm -f configs/*.last

remove-database:
	${CMD_DEL} statefulset/posdapostgresql
	${CMD_DEL} svc/posdapostgresql
	${CMD_DEL} secret/posda-postgresql-secret
	rm -f database.last
	rm -f database/*.last

remove-database-pvc:
	${CMD_DEL} pvc/posda-postgresql-pv-claim-auto
	rm -f pvcs/*.last

remove-database-secret:
	${CMD_DEL} secret/posda-postgresql-secret

remove-redis:
	${CMD_DEL} statefulset/redis
	${CMD_DEL} svc/redis
	rm -f redis.last
	rm -f redis/*.last

remove-pvc-posda-cache:
	${CMD_DEL} pvc/posda-cache-auto
	rm -f pvc/pvc-posda-cache.last

remove-posda-api:
	${CMD_DEL} deployment/posda-api
	${CMD_DEL} service/posda-api
	rm -f posda-api.last
	rm -f posda-api/*.last

remove-posda:
	${CMD_DEL} deployment/posda
	${CMD_DEL} service/posda
	rm -f posda.last
	rm -f posda/*.last

remove-ffp:
	${CMD_DEL} deployment/ffp
	rm -f ffp.last
	rm -f ffp/*.last

remove-web:
	${CMD_DEL} deployment/posdaweb
	${CMD_DEL} service/posdaweb
	rm -f web.last
	rm -f web/*.last

remove-worker-low:
	${CMD_DEL} deployment/worker-low
	rm -f worker-low.last
	rm -f worker/low.last

remove-worker-high:
	${CMD_DEL} deployment/worker-high
	rm -f worker-high.last
	rm -f worker/high.last

remove-kaleidoscope:
	${CMD_DEL} deployment/kaleidoscope
	${CMD_DEL} service/kaleidoscope
	rm -f kaleidoscope.last
	rm -rf kaleidoscope/*.last

remove-magicka:
	${CMD_DEL} deployment/magicka
	rm -f magicka.last
	rm -f magicka/*.last

