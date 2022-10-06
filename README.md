
import numpy as np
import pandas as pd
import streamlit as st
import requests

# getting url, accessing the data, and getting the specific list in the dictionary for performers
performers_url = "https://api.seatgeek.com/2/performers?client_id=MjYzMjg5MDB8MTY0ODU5MDg1My41NTk3NDM0"
performers_dict = requests.get(performers_url).json()
performers_list = performers_dict["performers"]

# assigning the indexes with the performer
eaglesBaseball = performers_list[0]
superBowl = performers_list[1]
wildcatsBaseball = performers_list[2]
playoffMLB = performers_list[3]
uniNotreDameFB = performers_list[4]
collegeFootballPlayoff = performers_list[5]
panthersBaseball = performers_list[6]
texasBowl = performers_list[8]
matteoTennis = performers_list[9]

#variables separating data into sports groups
baseball = [eaglesBaseball, wildcatsBaseball, playoffMLB, panthersBaseball]
football = [superBowl, uniNotreDameFB, collegeFootballPlayoff, texasBowl]
tennis = [matteoTennis]

#variable to group all the events
sports_events = [baseball, football, tennis]

# title and subheader
st.title("**Sport Events**")
st.subheader("*Your trusty search engine for the lovers of sports events*")

#asking user for name and not displaying info until name is entered
name = st.text_input("What's your name?")
if name:
    st.write("Welcome, ", name)

    sports = ["Baseball", "Football", "Tennis"]

    # variables for event names
    baseball_events = [eaglesBaseball["name"], wildcatsBaseball["name"],
                       playoffMLB["name"], panthersBaseball["name"]]
    football_events = [superBowl["name"], uniNotreDameFB["name"],
                       collegeFootballPlayoff["name"], texasBowl["name"]]
    tennis_events = [matteoTennis["name"]]

    # creating variable for columes: search, and checkboxes
    col1, col2,= st.columns(2)
    second_col1, second_col2 = st.columns(2)

    # method for map, takes in number to replace index and displays map
    def map(event):
        latitude = event["location"]["lat"]
        longitude = event["location"]["lon"]

        df = pd.DataFrame(
            np.random.randn(1, 1) / [50, 50] + [latitude, longitude], columns=['lat', 'lon']
        )

        st.map(df)

    # function to display info about each event
    def event_information(search):

        if search == "Baseball":
            events = baseball_events
            info = baseball

        elif search == "Football":
            events = football_events
            info = football
        elif search == "Tennis":
            events = tennis_events
            info = tennis

        with col2:
         choices = st.multiselect('Choose performers you want to view', events)

        if choices:
            for i in choices:
                st.write('**Selected:**', i)
                for j in info:
                    if j["name"] == i and j["has_upcoming_events"]:

                        num_of_events = j["num_upcoming_events"]
                        st.success("This performer has upcoming events.")
                        st.write("Number of events: ", num_of_events)

                        if j["location"]:
                            if st.button('View Location'):
                                location = map(j)
                        else:
                            st.error("Location unavailable.")

                        tickets = st.radio(
                            "Would you like to view tickets for upcoming events? ",
                            ('Yes', 'No'))

                        if tickets == 'Yes':
                            with st.expander("Click to view link"):
                                 st.write(j["url"])

                    elif j["name"] == i and not j["has_upcoming_events"]:
                        st.error("Sorry, there are no upcoming events for this performer.")


    # selectbox with 3 choices of sports to filter by
    with col1:
        search_options = st.selectbox("Search by sport ", (sports))

    # interactive table to see all events
    with second_col1:
        all_available = st.checkbox('See all performers available')

    if all_available:
        all_events = pd.DataFrame(
            {
                 "Baseball": [baseball_events[0], baseball_events[1], baseball_events[2], baseball_events[3]],
                 "Football": [football_events[0], football_events[1], football_events[2], football_events[3]],
                 "Tennis": [tennis_events[0], "", "", ""]
                }
         )
        st.dataframe(all_events)

    # showing 2 types of charts to display num of events
    with second_col2:
        compare_events = st.checkbox('Compare number of events among sports')

    if compare_events:

        baseball_count = 0
        football_count = 0
        tennis_count = 0

        for i in sports_events:
            for j in i:
                if j["taxonomies"][1]["name"] == "baseball":
                    baseball_count += j["num_upcoming_events"]
                elif j["taxonomies"][1]["name"] == "football":
                    football_count += j["num_upcoming_events"]
                elif j["taxonomies"][1]["name"] == "tennis":
                    tennis_count += j["num_upcoming_events"]

        chart_type = st.select_slider("Select Chart Type", ["Bar Chart", "Line Chart"])

        if (chart_type == "Bar Chart"):
            compare_events = pd.DataFrame([baseball_count, football_count, tennis_count], sports, columns=[
                "Number of Events"])
            st.bar_chart(data=compare_events, width=350, height=0, use_container_width=False)

        if (chart_type == "Line Chart"):
            compare_events = pd.DataFrame([baseball_count, football_count, tennis_count], sports,
                                            columns=["Number of Events"])
            st.line_chart(data=compare_events, width=350, height=0, use_container_width=False)


    # calling to event info function
    event_information(search_options)


