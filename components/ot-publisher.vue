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
    console.log("In mounted");
    console.log(this.publisher);
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
  },

    watch: {
    // whenever options changes, this function will run
    opts: function () {
      this.session.unpublish(this.publisher);
      this.publisher = OT.initPublisher(this.$el, this.opts, (err) => {
        if (err) {
          this.$emit('error', err);
        } else {
          this.$emit('publisherCompleted');
        }
      });
      console.log("In wathcer");
      console.log(this.publisher);

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
  },
}
</script>
