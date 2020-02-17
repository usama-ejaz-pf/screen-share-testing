<template>
  <div></div>
</template>

<script>
import OT from '@opentok/client'

export default {
  name: 'ot-publisher',

  props: {
    session: {
      type: OT.Session,
      required: false
    },
    opts: {
      type: Object,
      required: false
    }
  },

  data(){
    return{
      publisher: '',
    }
  },

  mounted: function() {

    this.publisher = OT.initPublisher(this.$el, this.opts, (err) => {
      if (err) {
        this.$emit('error', err);
      } else {
        this.$emit('publisherCompleted');
      }
    });

    this.publishSession();

  },

  watch: {
    // whenever options changes, this function will run
    opts: function () {
        this.session.unpublish(this.publisher);
        if(this.opts.videoSource === 'screen'){
            Promise.all([
                OT.getUserMedia({
                    videoSource: 'screen'
                }),
                OT.getUserMedia({
                    videoSource: null
                })
            ]).then(([screenStream, micStream]) => {
               this.publisher = OT.initPublisher(this.$el, {
                    videoSource: screenStream.getVideoTracks()[0],
                    audioSource: micStream.getAudioTracks()[0]
                },  (err) => {
                    if (err) {
                        this.$emit('error', err);
                    } else {
                        this.$emit('publisherCompleted');
                    }
                });

                this.publishSession();

            });
        }else {

            this.publisher = OT.initPublisher(this.$el, this.opts, (err) => {
                if (err) {
                    this.$emit('error', err);
                } else {
                    this.$emit('publisherCompleted');
                }
            });

            this.publishSession();
        }
    }
  },

  methods:{
      publishSession(){
          this.$emit('publisherCreated', this.publisher);
          const publish = () => {
              this.session.publish(this.publisher, (err) => {
                  if (err) {
                      this.$emit('error', err);
                  } else {
                      this.$emit('publisherConnected', this.publisher);
                  }
              });
          };
          if (this.session && this.session.isConnected()) {
              publish()
          }
          if (this.session) {
              this.session.on('sessionConnected', publish)
          }
      }
  }
}
</script>
