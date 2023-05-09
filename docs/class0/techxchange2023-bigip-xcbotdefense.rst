**Task 1 – Define the protected application in F5 Distributed Cloud Bot
Defense**

1. Login to the F5 Distributed Cloud tenant at
   `https://f5-xc-lab-sec.console.ves.volterra.io <https://f5-xc-lab-sec.console.ves.volterra.io/>`__.

..

   *The UDF deployment associated with this lab guide creates an
   ephemeral account and namespace for you in a F5 Distributed Cloud
   tenant. Shortly after launching this UDF deployment, you will receive
   an email requesting you to update/reset your password in this new
   ephemeral account. Please complete the password reset before
   continuing in this lab.*

   .. image:: images/media/image1.png
      :alt: A screenshot of a computer Description automatically
      generated
      :width: 3.68925in
      :height: 3.37788in

   *Example F5 Distributed Cloud ephemeral account Password Reset
   email.*

2. From the Home page select the Bot Defense tile.

.. image:: images/media/image2.png
   :alt: Graphical user interface, application, Teams Description
   automatically generated
   :width: 5.65675in
   :height: 4.27822in

3. From the left-hand menu select **Manage >> Applications**. Then
   select the “\ **Add Protected Application**\ ” button.

.. image:: images/media/image3.png
   :alt: Graphical user interface, application, Teams Description
   automatically generated
   :width: 5.99154in
   :height: 2.65715in

4. On the Protected Application screen do the following:

   a. Give the application a descriptive **Name** (like “f5bank”)

   b. Leave the **Application Region** as “US”.

   c. Select “Custom” for the **Connector Type**.

   d. Click **Save and Exit**.

.. image:: images/media/image4.png
   :alt: A screenshot of a computer Description automatically generated
   with medium confidence
   :width: 6.5in
   :height: 4.19514in

5. This will return you to the **Overview** screen.

From the left-hand menu select **Manage >> Applications** again. Click
the ellipsis icon on the right of your newly defined application. Here
you will find the ability to copy various values that are needed to
configure the BIGIP connector.

.. image:: images/media/image5.png
   :alt: A screenshot of a computer Description automatically generated
   with medium confidence
   :width: 6.5in
   :height: 2.55069in

**
**

**Task 2 – Configure BIGIP Distributed Cloud Bot Defense Profile**

1. Access the Web App in your UDF deployment.

.. image:: images/afbbbd283923ef793f053d33407b6c3d9801d6ad.png
   :width: 5.52083in
   :height: 1.71376in

2. Take note of the FQDN. You will need this when configuring the Bot
   Defense profile on the BIGIP.

.. image:: images/media/image7.png
   :width: 5in
   :height: 3.46875in

3. Access the TMUI of your BIGIP 17.1. You can find the credentials to
   login in the Details page.

.. image:: images/b802104b0df2d84619066c9c9a8f696ca5fac6ef.png
   :width: 5.44792in
   :height: 1.74787in

4. In the F5 BIGIP TMUI, browse to **Distributed Cloud Services >> Bot
   Defense >> BD Profiles** and click the (+) icon to create a new Bot
   Defense profile.

.. image:: images/media/image9.png
   :alt: Graphical user interface, text, application Description
   automatically generated
   :width: 4.68476in
   :height: 5.54063in

5. On the **New BD Profile…** screen edit the following settings:

..

   **General Properties**

a. Give the BD profile a descriptive **Name**.

..

   **API Request Settings**

b. Paste into the **Application ID** field the value copied from F5
   Distributed Cloud console.

c. Paste into the **Tenant ID** field the value copied from F5
   Distributed Cloud console.

d. Paste into the **API Key** field the value copied from F5 Distributed
   Cloud console.

.. image:: images/media/image10.png
   :alt: A screenshot of a computer Description automatically generated
   with medium confidence
   :width: 5.73387in
   :height: 6.1247in

   **JS Insertion Configuration**

e. Select the check box to enable **Inject JS in Specific URL**.

f. In the **JS Inject Included Paths**, enter **/login/** and click
   **Add**.

..

   **Protected Endpoint(s) – Web**

g. For **Protected URIs**:

   i. In the **Host** field paste in the FQDN from the Web App Access
      Method for your BIGIP.

..

   *(See Exercise 2 step 2 above. The FQDN for your Web App will be
   similar to 3995dde2-4cf8-4c5b-89f2-2d0717d76d5b.access.udf.f5.com.)*

ii.  Enter /**login/** into the **Path** field.

iii. For now, leave the **Mitigation Action** set to **Continue**.

..

   **NOTE:** *You will enable Blocking in a later step.*

iv. Click **Add**.

v.  Repeat steps i-iv above using **botdefense.udf.f5.com** in the
    **Host** field

..

   |A screenshot of a computer Description automatically generated with
   medium confidence|\ **NOTE:** *Ensure that both* **Hosts** *are
   listed in the* **Protected URIs** *section, as pictured above.*

   *The ephemeral hostname is needed to match requests that originate
   from outside the UDF environment. The botdefense.access.udf.com
   hostname is needed to match requests that originate from inside the
   UDF environment (as the ephemeral hostnames are not accessible from
   inside UDF).*

   **Advanced Features**

h. Select the **Advanced** view from the section dropdown.

i. From the **Protection Pool – Web** dropdown select the
   **ibd-webus.fastcache.net** pool.

j. From the **SSL Profile** dropdown select the **serverssl** profile.

k. Choose **X-Forwarded-For** from the **Source of Client IP Address**
   dropdown.

..

   .. image:: images/media/image12.png
      :alt: Graphical user interface, application Description
      automatically generated
      :width: 5.29463in
      :height: 3.86746in

l. Click **Finished**.

The F5 Distributed Cloud Bot Defense connector profile is now
configured. However, in order to protect the application we must assign
the BD profile to the virtual server.

6. From the F5 BIGIP TMUI, browse to **Local Traffic >> Virtual
   Servers**. Select the **app-virtual** virtual server.

.. image:: images/media/image13.png
   :alt: A screenshot of a computer Description automatically generated
   with medium confidence
   :width: 5.69645in
   :height: 3.35518in

a. Select the **Distributed Cloud Services** tab at the top and then do
   the following:

a. Set **Bot Defense** to **Enabled**.

b. From the **Profile** dropdown, select the BD profile created in the
   previous step.

c. Click **Update**.

..

   .. image:: images/8b78c94cd9106a8dbadf3cd321f1c9dbfde09d3c.png
      :alt: A screenshot of a server Description automatically generated
      with medium confidence
      :width: 5.84425in
      :height: 3.08134in

1. Clear all existing connections on the F5 BIGIP.

   a. Return to the UDF course tab in your browser and connect to the
      BIGIP using the Web Shell access method.

   b. Run the following command:

..

   **tmsh delete sys conn**

   NOTE: Clearing the connections is necessary to ensure that all
   requests to the virtual server are using the new configuration with
   the XC Bot Defense profile attached.

**Task 3 – Test Bots**

1. Connect to the Bot server in your UDF deployment using the Web Shell
   access method:

.. image:: images/8a76b2760362ac4bbe9a97ec9e929c3c7300b640.png
   :width: 5.54167in
   :height: 1.68559in

2. Change to the */home/ubuntu/bots* directory and list the contents:

   a. cd /home/ubuntu/bots

   b. ls

.. image:: images/media/image16.png
   :alt: A black background with white text Description automatically
   generated with low confidence
   :width: 4.87932in
   :height: 0.63789in

   There are 3 types of Bots available for this Lab and a README file
   where you can find detailed information on how to make them work if
   you are interested in using them elsewhere.

3. Change to the **advanced** directory.

   a. cd advanced

..

   In the **advanced** directory is a bot created using NodeJS and the
   Puppeteer browser automation tool.

   This Bot loads a Headless Chrome browser on stealth mode and attempts
   to log in using the credentials provided in the credentials/cred.txt
   file. Please open the credentials file and include the users you
   created during the 1st step of this lab instructions.

4. Run the **advanced** Bot by issuing the following command:

*node bot_multiple.js*

.. image:: images/media/image17.png
   :alt: A screen shot of a computer program Description automatically
   generated with low confidence
   :width: 5.39678in
   :height: 1.86696in

*If the Bot succeeds in sending the requests, you should get a similar
output as the one above.*

5. In the **medium** directory will you find a Bot created using Python
   and Selenium browser automation tool.

..

   This Bot loads a Headless Chrome browser and attempts to log in using
   the credentials provided in the usernames.txt and passwords.txt
   files. Please open these files and include the users you created
   during the 1st step of this lab instructions.

6. Run the **medium** Bot by following the instructions below:

a. Change to the medium directory.

b. Run the command: source .venv/bin/activate

..

   *(this will activate the python Virtual Environment)*

c. Run the command: python bot_medium.py

..

   .. image:: images/media/image18.png
      :alt: A picture containing text, screenshot, font Description
      automatically generated
      :width: 5.25408in
      :height: 2.28575in

   *If the Bot succeeds in sending the requests, you should get a
   similar output as the one above.*

7. In the **simple** directory will you find Bots created using *curl*
   and *python*.

..

   These Bots were created to replicate an automation tool that does not
   use a Web browser to send the requests and they should be detected by
   F5 Bot Defense as Token Missing request.

8. In order to run the **simple** Bots, please follow the instructions
   below:

a. Change to the simple directory.

b. Run the command: deactivate

..

   *(this will deactivate the previous python Virtual Environment)*

c. Run the command: source .venv/bin/activate

..

   *(this will activate the current python Virtual Environment)*

d. Run the command: ./curl_shape_token_missing.sh 10
   botdefense.udf.f5.com

   a. You should get the following output:

..

   .. image:: images/media/image19.png
      :alt: A picture containing text, screenshot, font Description
      automatically generated
      :width: 5.56135in
      :height: 1.16158in

e. Run the command: python py_token_missing.py

   a. You should get the following output:

..

   .. image:: images/media/image20.png
      :alt: A screenshot of a computer program Description automatically
      generated with medium confidence
      :width: 5.3429in
      :height: 4.92849in

9. **OPTIONAL:** Return to the BIGIP TMUI and change the configuration
   for the two Protected URIs to enable Blocking. Then re-run steps 3
   through 8 above.

**Task 4 – Review F5 Distributed Cloud Bot Defense Dashboard**

1. Return to the F5 Distributed Cloud Console.

You may be required to re-authenticate if you have not been on this page
for a while.

If you have already closed this browser tab you can login at
`https://f5-xc-lab-sec.console.ves.volterra.io <https://f5-xc-lab-sec.console.ves.volterra.io/>`__
and select the Bot Defense tile.

2. From the right-hand menu, select **Overview > Monitor** and change
   the time range to **Last 1 hour**.

Review the information on the **Monitor** dashboard.

*If it has been more than 1 hour since you started this lab you can
select a longer time range.*

.. image:: images/media/image21.png
   :alt: A screenshot of a computer Description automatically generated
   with medium confidence
   :width: 6.1279in
   :height: 3.44433in

3. From the right-hand menu, select **Report > Traffic Analyzer**.

.. image:: images/media/image22.png
   :alt: A picture containing screenshot, text Description automatically
   generated
   :width: 6.5in
   :height: 2.62292in

*On this page you can review details about individual requests.*

4. Add a filter to filter out the requests for the client JS.

   a. Select Add Filter

..

   .. image:: images/media/image23.png
      :alt: A screenshot of a computer Description automatically
      generated with medium confidence
      :width: 3.77951in
      :height: 2.10538in

b. Choose **Traffic Type**

c. Choose **Not In**

d. Select **Others**

e. Click **Apply**

..

   .. image:: images/media/image24.png
      :alt: A screenshot of a computer Description automatically
      generated with low confidence
      :width: 4.75381in
      :height: 2.06913in

5. From the right-hand menu, select **Report > Bad Bot Report**.

.. image:: images/media/image25.png
   :alt: A screenshot of a computer Description automatically generated
   with medium confidence
   :width: 5.36331in
   :height: 4.00644in

Review the information available on this page.

Be sure to scroll down to see all graphs and data available.

.. |A screenshot of a computer Description automatically generated with medium confidence| image:: images/media/image11.png
   :width: 6.23488in
   :height: 3.37656in
